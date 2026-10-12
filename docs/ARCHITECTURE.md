# Architecture V1

## Principe

Le projet doit être conçu **API-first / headless-first**.

Deux clients utilisent le même moteur :

```text
Dashboard autonome ─┐
                    ├──> Compliance API ──> moteur métier
Solenda ────────────┘
```

Aucune logique métier ne doit exister uniquement dans le dashboard.

## Flux général

```text
Faits client
+
Faits produit / sourcing / marketplace
        ↓
Normalisation & enrichissement
        ↓
Product Resolution
        ↓
Entity Resolution
        ↓
EPR Classification
        ↓
Official Registry Matching
        ↓
Route Generator
        ↓
Country / Supply-chain Rules
        ↓
OUI / NON
        ↓
Reasons + Evidence
```

## Faits client

Persistants :
- seller entity ;
- country of establishment ;
- VAT registration countries ;
- available prep centers ;
- optional Amazon/FBA facts explicitly known.

Important :
- le profil décrit des faits ;
- il ne contient pas une liste déclarative de pays « où je peux vendre/stocker » ;
- le moteur produit cette conclusion.

## Faits transactionnels

Par analyse :
- identifier ;
- sourcing / supplier stock country ;
- target marketplace ;
- optional supplier identity ;
- optional enriched product attributes ;
- optional Amazon/FBA facts from Solenda or providers.

## Route Generator

Le générateur produit des routes candidates raisonnables à partir des faits disponibles.

Exemple conceptuel :

```text
Supplier stock IT
→ Prep FR
→ FBA/marketplace DE
```

Le générateur ne doit :
- ni coder un trajet unique ;
- ni générer un produit cartésien arbitraire de tous les pays ;
- ni exiger que le client construise lui-même la route.

Une route peut être partielle si certains faits sont inconnus. Dans ce cas, le moteur conserve l'état interne UNVERIFIED et retourne NON à l'utilisateur si l'information manquante est nécessaire à la décision.

## Product Resolution

Entrée :
- EAN / GTIN / UPC / ASIN.

Sortie possible :
- GTIN ;
- ASIN ;
- title ;
- brand ;
- manufacturer ;
- category ;
- product_type ;
- image_url.

Amazon SP-API reste un provider optionnel.

## Entity Resolution

Distinguer :
- brand ;
- trademark owner ;
- GTIN licensee ;
- manufacturer ;
- parent group ;
- local entity ;
- distributor ;
- importer ;
- EPR registered producer.

## EPR Classification

Minimum V1 :
- PACKAGING
- EEE/WEEE
- BATTERY

L'architecture doit être extensible.

## Compliance Engine

Le moteur évalue :
- pays / marché cible ;
- filières applicables ;
- entité potentiellement responsable ;
- enregistrements officiels ;
- applicabilité de la couverture amont ;
- route physique connue ou déduite.

États internes :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

Sortie utilisateur :
- YES
- NO

## API

### Endpoint principal

```http
POST /api/v1/analyses
```

Le même endpoint accepte un payload minimal ou enrichi.

Exemple minimal :

```json
{
  "identifier": "4006381333931",
  "supplier_stock_country": "IT",
  "target_marketplace": "DE"
}
```

Exemple enrichi depuis Solenda :

```json
{
  "identifier": "4006381333931",
  "supplier_stock_country": "IT",
  "target_marketplace": "DE",
  "product": {
    "asin": "B0XXXX",
    "brand": "Example",
    "category": "Electric Toothbrush"
  },
  "supplier": {
    "legal_name": "Example Supplier",
    "country": "IT"
  },
  "account_context": {
    "seller_country": "FR",
    "vat_countries": ["FR", "DE"],
    "prep_centers": [
      {"country": "FR"}
    ]
  },
  "amazon_context": {
    "inventory_countries": ["DE"]
  }
}
```

Les champs enrichis sont facultatifs.

Le moteur doit :
- utiliser les données fournies ;
- éviter de recalculer ce qui est déjà fiable ;
- signaler les données manquantes réellement bloquantes ;
- ne jamais faire dépendre son fonctionnement de Solenda.

## Réponse

```json
{
  "analysis_id": "uuid",
  "verdict": "NO",
  "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE",
  "missing_context": [],
  "routes": [],
  "evidence": []
}
```

## Multi-tenant

Le tenant vient du contexte d'authentification.
Ne jamais faire confiance à un tenant_id libre fourni dans le body.

## Persistance

PostgreSQL.

Entités principales :
- users
- client_profiles
- client_vat_countries
- user_prep_centers
- products
- product_identifiers
- brands
- entities
- entity_aliases
- entity_relationships
- epr_streams
- epr_registrations
- product_epr_classifications
- analyses
- candidate_routes
- route_results
- evidence

Ne pas créer en V1 un modèle de données excessivement détaillé pour des notions FBA non encore nécessaires au moteur.

## Cache

- identifier → product
- brand → entity graph
- entity + country + stream → EPR registration
- normalized route facts → result, avec invalidation par version de règle/source

## Stack

- Next.js
- TypeScript
- React
- Tailwind
- shadcn/ui
- PostgreSQL
- Prisma
- Zod
- Vitest
- Playwright

Pas de microservices en V1.
