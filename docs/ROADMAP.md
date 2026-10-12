# Roadmap MVP V1

## Objectif

Valider le moteur REP/EPR comme brique indépendante avant intégration éventuelle à Solenda.

## Phase 0 — Spécification

- [x] positionnement : module de compliance, pas plateforme de sourcing
- [x] architecture API-first
- [x] dashboard autonome + Solenda sur le même moteur
- [x] Amazon optionnel
- [x] décision utilisateur OUI/NON
- [x] états internes conservateurs
- [x] distinction produit / entité / registre / couverture
- [x] profil client basé sur des faits
- [x] suppression de la logique « le client déclare ses pays FBA possibles »
- [x] REP/EPR déjà détenues non centrales en V1
- [ ] auditer toutes les sources officielles retenues
- [ ] valider les règles pays × filière
- [ ] constituer un jeu de cas de référence

## Gate avant développement du moteur

Avant d'autoriser un OUI réel :
- [ ] chaîne de preuve documentée ;
- [ ] règle pays/filière documentée ;
- [ ] données minimales nécessaires définies ;
- [ ] comportement UNVERIFIED défini ;
- [ ] corpus de test revu indépendamment.

## Phase 1 — Fondations

- [ ] Next.js / TypeScript
- [ ] PostgreSQL / Prisma
- [ ] auth
- [ ] isolation tenant
- [ ] API /api/v1/analyses
- [ ] modèle evidence
- [ ] provider abstraction
- [ ] identifier validation
- [ ] logs et observabilité

## Phase 2 — Profil client minimal

- [ ] seller entity
- [ ] country of establishment
- [ ] VAT registration countries
- [ ] available prep centers
- [ ] optional Amazon/FBA facts explicitly known

Ne pas demander :
- liste déclarative de pays FBA « possibles »
- routes construites manuellement
- REP/EPR personnelles comme donnée centrale

## Phase 3 — Product & Entity Resolution

- [ ] GTIN validation
- [ ] GS1 adapter si accès exploitable
- [ ] GLEIF
- [ ] EUIPO
- [ ] France Entreprises / RNE
- [ ] Companies House
- [ ] entity normalization
- [ ] corporate graph minimal

Amazon SP-API : optionnelle.

## Phase 4 — Classification REP

- [ ] PACKAGING
- [ ] EEE/WEEE
- [ ] BATTERY
- [ ] tests

## Phase 5 — Registres

Ordre proposé :
1. France
2. Allemagne
3. Royaume-Uni
4. Espagne
5. Italie

Pour chaque pays :
- [ ] source officielle
- [ ] ToS / automatisation
- [ ] normalisation
- [ ] cache/sync
- [ ] evidence
- [ ] tests

## Phase 6 — Route Generator

Entrées :
- stock fournisseur ;
- marketplace cible ;
- prep centers ;
- facts Amazon/FBA explicites si disponibles ;
- règles documentées.

Le moteur :
- [ ] génère les routes candidates raisonnables ;
- [ ] ne demande pas au client de construire la route ;
- [ ] n'infère pas un pays FBA uniquement depuis la TVA ;
- [ ] ne produit pas un produit cartésien arbitraire ;
- [ ] marque UNVERIFIED les routes dont une donnée nécessaire manque.

## Phase 7 — Compliance Engine

- [ ] règles pays/filières
- [ ] upstream coverage
- [ ] local entity resolution
- [ ] YES/NO
- [ ] reason codes
- [ ] missing_context
- [ ] evidence
- [ ] aucune note/score visible

## Phase 8 — Dashboard autonome

Champs d'analyse visés :
- [ ] identifier
- [ ] supplier/source stock country
- [ ] target marketplace

Données complémentaires :
- profil client ;
- enrichissement providers ;
- saisie complémentaire uniquement si réellement bloquante.

Afficher :
- [ ] produit
- [ ] routes candidates
- [ ] OUI/NON
- [ ] motifs
- [ ] filières
- [ ] entités
- [ ] numéros
- [ ] preuves
- [ ] limites

## Phase 9 — Intégration Solenda

- [ ] même endpoint que le dashboard
- [ ] payload enrichi facultatif
- [ ] données produit/sourcing/marketplace réutilisées
- [ ] contexte client transmis par Solenda
- [ ] aucun accès direct à la DB Solenda
- [ ] versioning API stable

## Gate d'intégration

Intégration possible si :
- décisions explicables ;
- règles auditées ;
- preuves traçables ;
- faux OUI maîtrisés ;
- providers suffisamment stables ;
- API autonome ;
- dashboard et Solenda produisent le même résultat sur le même payload.
