# Architecture V1

## Architecture produit

Le MVP doit être conçu **API-first / headless-first** afin de pouvoir être intégré ultérieurement dans Solenda. Le dashboard V1 sert essentiellement à tester et expliquer le moteur.

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
Supplier stock → prep center(s) → FBA inventory country/countries
        ↓
Amazon target marketplace(s)
        ↓
EPR Classification
Product facts → filière and applicable categories
        ↓
Official Registry Matching
        ↓
Supply-chain / Country Rules
        ↓
EPR_STATUS per route × target market × stream
        ↓
GO / HOLD for this EPR check only
        ↓
Evidence + Reason Codes
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

Modéliser séparément notre société vendeuse sur Amazon, le fournisseur/grossiste, l'opérateur du prep center, Amazon comme place de marché/fulfillment, et l'entité amont prétendue responsable. Une déclaration d'IDU ne suffit pas : conserver la preuve qui relie l'entité amont au produit/SKU, à la filière/catégorie et au marché concerné.

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
  "product_id": "...",
  "seller_entity_id": "...",
  "existing_epr_evidence_ids": ["..."],
  "product_facts": {
    "category_attributes": {},
    "packaging_attributes": {},
    "embedded_battery": null
  },
  "sales_channel": {
    "marketplace": "AMAZON",
    "fulfillment": "FBA"
  },
  "routes": [
    {
      "supplier_entity_id": "...",
      "supplier_establishment_country": "DE",
      "supplier_stock_country": "DE",
      "prep_center_countries": ["FR"],
      "amazon_fulfillment_program": "...",
      "fba_inventory_countries": ["FR"],
      "target_markets": ["FR", "DE", "IT", "ES", "GB"],
      "claimed_responsible_entity_id": "...",
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
  "epr_status": "UNVERIFIED",
  "registration_assessment": {
    "outcome": "UNRESOLVED",
    "entity_to_register": null,
    "responsible_entity_id": null,
    "evidence_ids": []
  },
  "action": "HOLD",
  "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE",
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

États possibles de `epr_status` : `VERIFIED`, `NON_COMPLIANT`, `UNVERIFIED`, `CONFLICT`. `HOLD` indique seulement que le parcours REP/EPR analysé ne doit pas être considéré comme vérifié ; il ne signifie pas à lui seul que la vente est interdite. Ne pas exposer `sell_allowed` ou `stock_allowed` comme conclusions générales sans couvrir les règles correspondantes.

`registration_assessment.outcome` répond à la question métier : `SELLER_REGISTRATION_MISSING`, `SELLER_REGISTRATION_VERIFIED`, `UPSTREAM_COVERAGE_VERIFIED`, `NO_SELLER_REGISTRATION_REQUIRED` ou `UNRESOLVED`. Retourner l'entité et les preuves justifiant l'issue. Ne pas déclarer `UPSTREAM_COVERAGE_VERIFIED` sans règle de marché vérifiée et lien documenté avec le produit/parcours.

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

Entités principales :
- users
- user_prep_centers
- seller_profiles
- supplier_profiles
- supply_routes
- route_segments
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

Une actual route est une chaîne fournie ou observée pour un produit, pas une combinaison théorique créée en croisant tous les pays. Si un utilisateur demande un marché cible pour lequel aucune règle n'est validée, renvoyer UNVERIFIED plutôt que de le retirer de la matrice.

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
