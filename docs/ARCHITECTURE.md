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
Local Entity Resolution
FR | DE | IT | ES | GB
        ↓
EPR Classification
PACKAGING | EEE/WEEE | BATTERY
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

### EPR Classification
Minimum V1 :
- PACKAGING
- EEE
- BATTERY

Prévoir une architecture extensible pour d'autres filières sans les implémenter dans la V1.

### Compliance Engine
Entrée :
```json
{
  "product_id": "...",
  "supplier_stock_country": "DE",
  "target_country": "FR"
}
```

Sortie :
```json
{
  "country": "FR",
  "verdict": "NO",
  "sell_allowed": false,
  "stock_allowed": false,
  "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE",
  "summary": "..."
}
```

## Règles minimales

- une filière obligatoire non vérifiée => NON ;
- entité locale non résolue => NON ;
- registre indisponible sans cache récent => NON ;
- une marque enregistrée ne suffit pas à prouver la couverture ;
- un numéro d'un autre pays ne vaut pas couverture locale ;
- l'achat dans un pays ne prouve pas la conformité du pays de destination ;
- UNKNOWN ≠ COMPLIANT.

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

Préférer les imports bulk officiels lorsqu'ils existent :

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
