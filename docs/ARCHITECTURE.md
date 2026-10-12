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
Seller / responsible entity + sales-path resolution
        ↓
Pilot: France / EEE
        ↓
EPR Classification
Product facts → filière and applicable categories
        ↓
Official Registry Matching
        ↓
Supply-chain / Country Rules
        ↓
OUI / NON
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
  "evaluated_entity_id": "...",
  "responsible_entity_id": "...",
  "seller_role": "...",
  "sales_channel": "...",
  "supplier_stock_country": "DE",
  "target_country": "FR"
}
```

Sortie :
```json
{
  "country": "FR",
  "epr_status": "UNVERIFIED",
  "action": "HOLD",
  "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE",
  "scope": {
    "stream": "EEE",
    "rule_version": "...",
    "evaluated_at": "..."
  },
  "summary": "..."
}
```

États possibles de `epr_status` : `VERIFIED`, `NON_COMPLIANT`, `UNVERIFIED`, `CONFLICT`. `HOLD` indique seulement que le parcours REP/EPR analysé ne doit pas être considéré comme vérifié ; il ne signifie pas à lui seul que la vente est interdite. Ne pas exposer `sell_allowed` ou `stock_allowed` comme conclusions générales sans couvrir les règles correspondantes.

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

## Persistance

PostgreSQL.

Entités principales :
- users
- user_prep_centers
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
- country_analysis_results
- evidence

## Cache

Réutiliser les résolutions existantes :
- GTIN → product
- ASIN → product
- brand → entity graph
- brand + country → local entity
- entity + country + stream → EPR registration

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
