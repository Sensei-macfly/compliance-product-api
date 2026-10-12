# Architecture V1

## Architecture produit

Le MVP est **API-first / headless-first** : son dashboard et les applications clientes tierces appellent le même moteur avec des données enrichies. Les dossiers et configurations sont isolés par client/tenant ; Solenda pourra aussi l'intégrer ultérieurement.

## Pipeline

```text
EAN / GTIN / UPC / ASIN
        ↓
Identifier Detection & Validation
        ↓
Product Resolution
        ↓
Brand Resolution
        ↓
Entity Resolution Engine
        ↓
Corporate Graph
        ↓
Seller / supplier / claimed responsible entity resolution
        ↓
Client profile: VAT countries, available prep centers, FBA programs/countries
        ↓
Sourcing/stock options + requested target marketplaces
        ↓
Candidate Route Generator (supported transitions only; no hard-coded route)
        ↓
EPR Classification
Product facts → filière and applicable categories
        ↓
Official Registry Matching
        ↓
Supply-chain / Country Rules
        ↓
Internal EPR_STATUS + registration assessment per route × target market × stream
        ↓
Binary user decision: OUI / NON
        ↓
Plain-language explanation + auditable evidence
```

## Modules

### Product Resolution
Normaliser l'identifiant et enrichir :
- GTIN ;
- ASIN si connu ;
- titre ;
- marque ;
- fabricant ;
- catégorie ;
- product type ;
- image éventuelle.

Un GTIN/ASIN ne décrit pas nécessairement les composants d'emballage, une batterie intégrée, un lot ou une variante de marché. Collecter ou vérifier les attributs nécessaires à chaque classification ; sinon retourner UNVERIFIED.

Les sources Amazon sont optionnelles.

### Entity Resolution
Distinguer :
- marque ;
- propriétaire de marque ;
- titulaire GTIN ;
- fabricant ;
- société mère ;
- filiale locale ;
- distributeur ;
- importateur ;
- producteur REP enregistré.

### Corporate Graph
Relations prévues :
- BRAND_OWNED_BY
- GTIN_LICENSED_TO
- MANUFACTURED_BY
- SUBSIDIARY_OF
- PARENT_OF
- DISTRIBUTED_BY
- LOCAL_ENTITY_OF
- REGISTERED_EPR_AS

Chaque relation conserve sa source et sa date de vérification.

Ajouter une relation entre l'entité qui doit remplir l'obligation et le parcours produit/commercial évalué. Ne pas déduire cette relation de la seule marque, du propriétaire du GTIN ou d'une société du même groupe.

Modéliser séparément l'entité vendeuse du client, le fournisseur/grossiste, l'opérateur du prep center, Amazon comme place de marché/fulfillment, et l'entité amont prétendue responsable. Une déclaration d'IDU ne suffit pas : conserver la preuve qui relie l'entité amont au client, au produit/SKU, à la filière/catégorie et au marché concerné.

### Candidate Route Generator

Générer les routes candidates à partir des sources/lieux de stock, du profil client, du produit, des prep centers disponibles, des contraintes FBA et des marketplaces demandées. Ne pas coder de route fixe ni énumérer des permutations sans transitions logistiques étayées. Les pays TVA peuvent être des contraintes d'entrée uniquement lorsqu'une règle de routage l'établit ; ils ne prouvent ni l'éligibilité FBA ni la conformité REP/EPR. Si le programme ou les pays d'inventaire FBA ne sont pas connus, ne pas les déduire du marché cible : garder la route en NON avec état interne UNVERIFIED.

### EPR Classification
Minimum V1 :
- PACKAGING
- EEE
- BATTERY

Prévoir une architecture extensible pour d'autres filières sans les implémenter dans la V1.

### Compliance Engine
Entrée minimale à valider pour chaque parcours :
```json
{
  "identifier": "8710103995426",
  "client_configuration": {
    "seller_entity_id": "...",
    "existing_epr_evidence_ids": ["..."],
    "vat_registration_countries": ["FR", "DE"],
    "available_prep_centers": [
      {"id": "prep-1", "country": "FR", "eligible_product_types": ["..."]}
    ],
    "amazon_fulfillment_profiles": [
      {"program": "...", "eligible_inventory_countries": ["FR", "DE"]}
    ],
    "supported_logistics_transitions": [
      {"from_stock_country": "DE", "via_prep_center_id": "prep-1", "to_fba_country": "FR"}
    ]
  },
  "sourcing_options": [
    {
      "supplier_entity_id": "...",
      "supplier_establishment_country": "DE",
      "available_stock_countries": ["DE", "PL"]
    }
  ],
  "target_markets": ["FR", "DE", "IT", "ES", "GB"],
  "product_facts": {
    "category_attributes": {},
    "packaging_attributes": {},
    "embedded_battery": null
  },
  "upstream_claims": [
    {
      "claimed_responsible_entity_id": "...",
      "product_or_sku_scope": "...",
      "stream": "EEE",
      "evidence_ids": ["..."]
    }
  ]
}
```

Sortie :
```json
{
  "route_id": "...",
  "target_market": "FR",
  "decision": "NON",
  "explanation": "Les preuves disponibles ne démontrent pas que la route peut être poursuivie sans nouvel enregistrement.",
  "internal": {
    "epr_status": "UNVERIFIED",
    "registration_assessment": {
      "outcome": "UNRESOLVED",
      "entity_to_register": null,
      "responsible_entity_id": null,
      "evidence_ids": []
    },
    "action": "HOLD",
    "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE"
  },
  "scope": {
    "supplier_stock_country": "DE",
    "prep_center_countries": ["FR"],
    "fba_inventory_countries": ["FR"],
    "stream": "EEE",
    "rule_version": "...",
    "evaluated_at": "..."
  },
  "summary": "..."
}
```

États internes possibles de `epr_status` : `VERIFIED`, `NON_COMPLIANT`, `UNVERIFIED`, `CONFLICT`. L'interface ne montre que `decision: OUI | NON` : OUI seulement si l'état interne est VERIFIED et la situation d'inscription est favorablement établie ; NON dans tous les autres cas. `HOLD` et les états détaillés restent internes. Un NON fondé sur une incertitude ne signifie ni interdiction juridique ni preuve qu'une inscription propre est obligatoire. Ne pas exposer `sell_allowed` ou `stock_allowed` comme conclusions générales sans couvrir les règles correspondantes.

`internal.registration_assessment.outcome` répond en interne à la question métier : `SELLER_REGISTRATION_MISSING`, `SELLER_REGISTRATION_VERIFIED`, `UPSTREAM_COVERAGE_VERIFIED`, `NO_SELLER_REGISTRATION_REQUIRED` ou `UNRESOLVED`. Retourner l'entité et les preuves justifiant l'issue ; afficher une explication en langage clair, sans transformer ces états en décisions visibles supplémentaires. Ne pas déclarer `UPSTREAM_COVERAGE_VERIFIED` sans règle de marché vérifiée et lien documenté avec le produit/parcours.

## Règles minimales

- filière ou catégorie non vérifiée => UNVERIFIED ;
- entité ou rôle dans le parcours non résolu => UNVERIFIED ;
- registre indisponible sans cache à la fraîcheur définie => UNVERIFIED ;
- une marque enregistrée ne suffit pas à prouver la couverture ;
- un numéro d'un autre pays ne vaut pas couverture locale ;
- l'achat dans un pays ne prouve pas la conformité du pays de destination ;
- UNKNOWN ≠ COMPLIANT.
- une absence dans le registre n'est NON_COMPLIANT que si l'exhaustivité, l'actualité de la source et l'identité de l'entité ont été établies ; sinon UNVERIFIED.
- un IDU officiel prouve l'enregistrement de l'entité pour une filière donnée, mais ne lie pas à lui seul cet enregistrement à un GTIN, un vendeur ou un contrat.
- un changement de prep center, de pays de stock FBA, de programme FBA ou de marché cible crée un contexte d'analyse distinct ;
- un pays d'inventaire FBA manquant ou un programme non renseigné => UNVERIFIED si pertinent pour la règle évaluée ;
- une inscription amont n'est pas une dérogation automatique pour le vendeur ; le moteur doit appliquer la règle du marché et vérifier le lien fournisseur-produit.

## Persistance

PostgreSQL.

Toutes les données client, options de sourcing, configurations logistiques, routes générées, analyses et preuves doivent être isolées par tenant. Résoudre le tenant depuis l'authentification et appliquer le contrôle d'accès côté API et persistance.

Entités principales :
- users
- client_profiles
- client_vat_countries
- user_prep_centers
- client_fba_profiles
- seller_profiles
- supplier_profiles
- sourcing_options
- supply_routes
- route_segments
- candidate_routes
- products
- product_identifiers
- brands
- entities
- entity_aliases
- entity_relationships
- brand_entity_relationships
- epr_streams
- epr_registrations
- product_epr_classifications
- analyses
- route_market_stream_results
- registration_assessments
- evidence

## Cache

Réutiliser les résolutions existantes :
- GTIN → product
- ASIN → product
- brand → entity graph
- entity + country + stream + category → EPR registration
- seller + product + actual route + target market + stream + rule version → analysis result

Les routes candidates sont générées depuis les données d'un client et des transitions logistiques/FBA étayées, pas à partir d'une route codée en dur ni d'un produit cartésien de pays. Un changement de profil, de source, de prep center ou de configuration FBA invalide les résultats générés. Si une marketplace demandée n'a pas de règle validée, produire NON et conserver UNVERIFIED en interne plutôt que de la retirer de la matrice.

## Synchronisation

Préférer les imports bulk officiels lorsqu'ils existent. Pour le candidat France / EEE, l'ADEME fournit une API publique et un export complet actualisé quotidiennement ; faire une synchronisation locale plutôt qu'un appel au registre par analyse :

```text
Official Registry
      ↓
Scheduled Sync
      ↓
Normalizer
      ↓
PostgreSQL
      ↓
Fast Local Lookup
```

## Stack recommandée

- Next.js
- React
- TypeScript
- Tailwind
- shadcn/ui
- PostgreSQL
- Prisma
- Zod
- Vitest
- Playwright

Éviter les microservices en V1.
