# Roadmap MVP V1

## Objectif

Valider séparément la faisabilité technique et réglementaire du moteur REP/EPR avant toute intégration à Solenda.

## Phase 0 — Spécification

- [x] positionnement MVP autonome
- [x] UX 2 champs
- [x] décision binaire OUI/NON
- [x] Amazon optionnel
- [x] APIs gratuites/officielles en priorité
- [ ] audit final des APIs et ToS
- [ ] shortlist de bibliothèques GitHub réellement utiles

## Phase 1 — Fondations

- [ ] Next.js / TypeScript
- [ ] PostgreSQL / Prisma
- [ ] schéma de données
- [ ] auth simple
- [ ] validation identifiants EAN/GTIN/UPC/ASIN
- [ ] système de providers
- [ ] audit logging / evidence model

## Phase 2 — Product & Entity Resolution

- [ ] GTIN validation
- [ ] GS1 adapter si accès disponible
- [ ] GLEIF adapter
- [ ] EUIPO adapter
- [ ] France Entreprises/RNE adapter
- [ ] Companies House adapter
- [ ] entity normalization / aliasing
- [ ] corporate graph minimal

Amazon SP-API : **optionnelle**, hors chemin critique.

## Phase 3 — Classification REP

- [ ] PACKAGING
- [ ] EEE/WEEE
- [ ] BATTERY
- [ ] tests de classification

## Phase 4 — Registres prioritaires

Ordre recommandé :
1. France
2. Allemagne
3. Royaume-Uni
4. Espagne
5. Italie

Pour chaque pays :
- [ ] provider officiel
- [ ] normalisation
- [ ] cache local
- [ ] evidence
- [ ] tests
- [ ] reason codes

## Phase 5 — Compliance Engine

- [ ] règles pays/filières
- [ ] YES/NO
- [ ] sell_allowed
- [ ] stock_allowed
- [ ] reason codes
- [ ] comportement fail-safe
- [ ] aucun score visible

## Phase 6 — Dashboard de test

- [ ] champ identifiant
- [ ] pays stock fournisseur
- [ ] fiche produit
- [ ] cartes FR/DE/IT/ES/GB
- [ ] OUI/NON
- [ ] obligations détectées
- [ ] entités et numéros
- [ ] sources/preuves
- [ ] limites explicites

## Phase 7 — Validation MVP

Tester sur un corpus de produits représentatifs :
- produit emballage seul ;
- EEE sans batterie ;
- EEE avec batterie ;
- grande marque internationale ;
- marque locale ;
- private label ;
- produit introuvable ;
- flux transfrontalier ;
- registre indisponible.

## Gate d'intégration Solenda

Le MVP n'est candidat à l'intégration que si :
- les décisions sont explicables ;
- les sources sont auditables ;
- le taux de faux OUI est maîtrisé ;
- les providers gratuits sont suffisamment stables ;
- le coût opérationnel est acceptable ;
- le moteur est utilisable via API indépendante du dashboard.
