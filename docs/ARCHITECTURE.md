# Architecture V1

## Principe

Architecture **API-first / headless-first**.

```text
Dashboard autonome ─┐
                    ├──> Compliance API ──> moteur métier
Solenda ────────────┘
```

Aucune logique métier ne doit exister uniquement dans le dashboard.

## Simplification structurante

La V1 n'a **pas de Route Generator**.

Elle ne modélise pas :
- transit ;
- prep centers ;
- stockage FBA ;
- redistribution Amazon.

La suppression du Route Generator est une simplification de représentation, pas une simplification de la preuve réglementaire.

### Safety gate des faits requis

Chaque règle `market × stream × role` doit déclarer ses faits réglementaires requis.

Si la détermination du rôle, de l'applicabilité ou de la couverture nécessite un fait non disponible ou hors scope V1, le moteur doit faire fail-closed :
- `UNVERIFIED` / `UNRESOLVED` ;
- verdict filière NON ;
- overall NON.

Aucun pays FBA, rôle d'importateur, première mise sur le marché, absence de reconditionnement ou autre fait favorable ne peut être inventé pour permettre OUI.

`product_handling_mode` fait partie des faits transactionnels de sécurité. `ORIGINAL_UNMODIFIED` doit venir d'une attestation explicite ou d'un fait fiable et ne possède aucun défaut favorable. Valeur absente/UNKNOWN/BUNDLED/REPACKAGED => pas de OUI V1.

Le cœur d'analyse est :

```text
Produit
+ vendeur
+ pays physique du stock fournisseur
+ marché cible
        ↓
Product Resolution
        ↓
Entity Resolution
        ↓
EPR Classification
        ↓
Target-market Rules
        ↓
Official Registry Matching
        ↓
Upstream Coverage Assessment
        ↓
Stream Results
        ↓
OUI / NON
```

## Faits persistants client

- seller legal entity ;
- country of establishment ;
- VAT registration countries si disponibles.

Les immatriculations REP propres du vendeur ne sont pas une voie positive recherchée par la V1.

## Faits transactionnels

- identifier ;
- supplier stock country ;
- target marketplace ;
- product handling mode ;
- optional supplier identity ;
- optional enriched product attributes.

### Définition de supplier_stock_country

Le pays où le produit est physiquement stocké chez le fournisseur au moment du sourcing.

Ne pas le déduire du pays de facture ou du siège social du fournisseur.

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

Amazon SP-API peut enrichir cette étape mais reste optionnelle.

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

Univers V1 :
- PACKAGING
- EEE/WEEE
- BATTERY

Chaque analyse contient une entrée pour chacune de ces filières.

`applicability` :
- APPLICABLE
- NOT_APPLICABLE
- UNRESOLVED

`stream_scope_state` :
- COMPLETE
- UNRESOLVED

## Upstream Coverage Assessment

Pour une filière applicable, le moteur doit relier :

```text
produit
→ marque / fabricant / entité amont
→ rôle réglementaire pertinent
→ registre officiel du marché cible
→ numéro / enregistrement
→ scope de l'enregistrement
→ couverture du produit / catégorie / marque
```

Un numéro officiel peut être vérifié tout en laissant la couverture produit UNPROVEN.

Seul `UPSTREAM_VERIFIED` permet un OUI sur une filière applicable.

`UPSTREAM_VERIFIED` n'est atteignable que si tous les faits requis par la règle activée sont établis et si la chaîne `evidence[]` est non vide et traçable. Une filière APPLICABLE avec OUI et `evidence=[]` est invalide par contrat. Si une règle nécessite un fait logistique/rôle exclu du modèle V1 et que ce fait n'est pas prouvé, la couverture reste UNPROVEN/UNRESOLVED.

## Verdict

Sortie utilisateur :
- OUI
- NON

États internes :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

Agrégation :
- NOT_APPLICABLE => OUI pour la filière ;
- APPLICABLE + UPSTREAM_VERIFIED => OUI ;
- APPLICABLE + autre statut => NON ;
- UNRESOLVED => NON ;
- scope global UNRESOLVED => overall NON ;
- overall OUI uniquement si toutes les filières V1 sont OUI.

## API

```http
POST /api/v1/analyses
```

Une requête V1 cible une marketplace.

Le contrat JSON normatif unique est dans `docs/CDC_V1.md`.

## Persistance

PostgreSQL.

Entités principales :
- users
- client_profiles
- client_vat_countries
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
- analysis_stream_results
- registry_matches
- evidence

Supprimés du modèle V1 :
- user_prep_centers
- candidate_routes
- route segments
- FBA inventory scenarios
- seller-owned EPR registration as positive path

## Cache

- identifier → product
- brand/product → entity graph
- entity + target market + stream → registration matches
- normalized product/market facts → analysis result, versionné par règles/sources

## UX contract

L'API doit retourner assez d'information pour que l'interface affiche :
- verdict immédiat ;
- tous les numéros officiels pertinents trouvés ;
- distinction « numéro vérifié » / « couverture produit vérifiée » ;
- éléments validés même si overall NON ;
- cause bloquante en français ;
- preuves/sources.

Les codes internes restent en anglais.

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
