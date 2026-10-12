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
- transit simple ;
- prep center ;
- FBA inventory country ;
- target marketplace.

Types de segments V1 :
- `SUPPLIER_STOCK`
- `TRANSIT`
- `PREP_CENTER`
- `FBA_INVENTORY`
- `MARKETPLACE`

Les segments sont ordonnés. `TRANSIT` et `PREP_CENTER` peuvent être répétés. Un segment `PREP_CENTER` doit référencer un `prep_center_id` du profil client, pas seulement un pays.

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

Univers minimum V1 :
- PACKAGING
- EEE/WEEE
- BATTERY

Pour chaque route, toutes les filières de cet univers sont représentées dans `streams[]` avec un état d'applicabilité :
- APPLICABLE
- NOT_APPLICABLE
- UNRESOLVED

Une filière UNRESOLVED force NON. Le moteur conserve aussi un `stream_scope_state` au niveau de la route ; si le périmètre global des filières n'est pas démontré complet, la route ne peut pas être OUI.

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

En V1, une requête porte sur **une seule marketplace cible**. Le dashboard ou Solenda répète l'appel pour plusieurs marketplaces ; un batch pourra être ajouté ultérieurement.

Le **contrat normatif unique** de requête et de réponse est défini dans `docs/CDC_V1.md`, section « Contrat API V1 de référence ». Ce document d'architecture ne maintient pas un second schéma JSON afin d'éviter toute divergence documentaire.

Invariants d'architecture :
- toutes les filières V1 sont présentes dans `streams[]` pour chaque route ;
- une applicabilité UNRESOLVED empêche un OUI ;
- un `stream_scope_state` UNRESOLVED empêche un OUI ;
- les routes utilisent des segments ordonnés et typés ;
- un prep center généré référence un centre réellement disponible via `prep_center_id` ;
- un transit sans prep center peut être représenté par `TRANSIT` ;
- un scénario de stockage FBA distinct produit une route candidate distincte ;
- `internal_state`, `applicability` et `stream_scope_state` restent techniques ;
- `overall_verdict = OUI` signifie qu'au moins une route complète est favorable pour la marketplace demandée.

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
