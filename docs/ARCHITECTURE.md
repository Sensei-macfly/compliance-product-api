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
Résultats route × marché × filière
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
- optional Amazon/FBA facts explicitly known ;
- optional existing seller EPR/REP/IDU registrations.

Important :
- le profil décrit des faits ;
- il ne contient pas une liste déclarative de pays « où je peux vendre/stocker » ;
- le moteur produit cette conclusion ;
- les immatriculations REP existantes sont facultatives, mais exploitables lorsqu'elles existent.

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
→ FBA DE
→ Marketplace DE
```

Le générateur ne doit :
- ni coder un trajet unique ;
- ni générer un produit cartésien arbitraire de tous les pays ;
- ni exiger que le client construise lui-même la route.

Le modèle préserve distinctement :
- supplier stock ;
- prep/transit ;
- FBA inventory country ;
- target marketplace.

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

Le moteur évalue, **par route × marché cible × filière** :
- filière applicable ;
- entité potentiellement responsable ;
- enregistrement officiel ;
- éventuelle inscription propre déjà valide du vendeur ;
- applicabilité de la couverture amont ;
- route physique connue ou déduite ;
- qualité et fraîcheur des preuves.

États internes :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

Sortie utilisateur :
- OUI
- NON

Un verdict global peut exister comme agrégat, mais ne remplace jamais les résultats détaillés.

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
    ],
    "existing_epr_registrations": [
      {
        "country": "DE",
        "stream": "PACKAGING",
        "registration_number": "OPTIONAL"
      }
    ]
  },
  "amazon_context": {
    "inventory_countries": ["DE"]
  }
}
```

Les champs enrichis sont facultatifs.

## Réponse

Le contrat API canonique est défini dans `docs/CDC_V1.md` et doit rester identique ici.

```json
{
  "analysis_id": "uuid",
  "overall_verdict": "OUI",
  "routes": [
    {
      "route_id": "route-1",
      "route_verdict": "OUI",
      "segments": [
        {"type": "SUPPLIER_STOCK", "country": "IT"},
        {"type": "PREP_CENTER", "country": "FR"},
        {"type": "FBA_INVENTORY", "country": "DE"},
        {"type": "MARKETPLACE", "country": "DE"}
      ],
      "target_marketplace": "DE",
      "streams": [
        {
          "stream": "PACKAGING",
          "verdict": "OUI",
          "internal_state": "VERIFIED",
          "reason_codes": ["ALL_REQUIRED_STREAMS_VERIFIED"],
          "missing_context": [],
          "evidence": []
        }
      ]
    }
  ]
}
```

Agrégation :
- une filière est OUI uniquement si elle est favorablement vérifiée ;
- une route est OUI uniquement si toutes ses filières applicables requises sont OUI ;
- `overall_verdict` est OUI si au moins une route candidate possède `route_verdict = OUI`, sinon NON.

Le verdict global signifie donc « au moins une route démontrée existe » et non « toutes les routes sont favorables ».

`internal_state` est purement technique et ne doit pas apparaître comme une troisième décision commerciale dans l'interface utilisateur.

## Sémantique de NON

Un NON dû à UNVERIFIED signifie :
> le moteur ne peut pas démontrer que cette combinaison route × marché × filière évite un nouvel enregistrement.

Cela ne constitue pas automatiquement :
- une interdiction de vente ;
- une preuve qu'une immatriculation est obligatoire ;
- une conclusion juridique définitive.

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
- seller_epr_registrations
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
- route_market_stream_results
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
