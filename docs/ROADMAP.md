# Roadmap MVP V1

## Objectif

Valider séparément la faisabilité technique et réglementaire du moteur REP/EPR avant toute intégration à Solenda.

## Phase 0 — Spécification

- [x] positionnement MVP autonome
- [ ] valider les informations nécessaires sur l'entité, son rôle et le parcours de vente
- [x] définir les états VÉRIFIÉ / NON CONFORME / NON VÉRIFIÉ / CONTRADICTOIRE
- [x] définir le standard provisoire d'une preuve positive et les conditions d'une absence concluante
- [x] UX proposée à deux champs, mais non suffisante pour établir seule un résultat REP/EPR
- [x] Amazon optionnel
- [x] APIs gratuites/officielles en priorité
- [x] vérifier une source candidate pour le pilote : API publique ADEME/SYDEREP France / EEE
- [ ] auditer les règles d'utilisation, la sémantique des champs et les limites de toutes les sources retenues
- [ ] shortlist de bibliothèques GitHub réellement utiles
- [ ] constituer un jeu de référence indépendant et fixer les critères de lancement avant évaluation

### Gate avant développement du moteur

- [ ] figer un parcours de vente et l'acteur dont on évalue l'obligation ;
- [ ] faire valider la portée REP/EPR et les règles du parcours France / EEE par une personne compétente ;
- [ ] produire une chaîne de preuves de bout en bout ; l'IDU ADEME seul ne relie pas une société au produit/GTIN ni au vendeur ;
- [ ] décrire la stratégie de synchronisation, l'attribution de la source, la fraîcheur et les cas d'indisponibilité ;
- [ ] définir les mesures de fiabilité positive, de couverture et de résultats indéterminés sur des cas de référence tenus à l'écart des règles.

Si cette gate échoue, ne pas masquer le problème par des verdicts NON générés à partir d'une absence de preuve ; réduire ou revoir la promesse du pilote.

## Phase 1 — Fondations

- [ ] commencer par le périmètre pilote uniquement : France / EEE ;
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
1. France / EEE (pilote de preuve d'enregistrement, pas autorisation produit de revente)
2. autres filières France après validation du pilote
3. Allemagne
4. Royaume-Uni
5. Espagne
6. Italie

Pour chaque pays :
- [ ] provider officiel
- [ ] normalisation
- [ ] cache local
- [ ] evidence
- [ ] tests
- [ ] reason codes

## Phase 5 — Compliance Engine

- [ ] règles pays/filières
- [ ] états VÉRIFIÉ / NON CONFORME / NON VÉRIFIÉ / CONTRADICTOIRE
- [ ] action HOLD distincte d'une conclusion sur le droit général de vendre
- [ ] reason codes
- [ ] comportement fail-safe
- [ ] aucun score visible

## Phase 6 — Dashboard de test

- [ ] champ identifiant
- [ ] pays stock fournisseur
- [ ] fiche produit
- [ ] vue limitée au pilote France / EEE
- [ ] raisons et preuves pour chaque état
- [ ] obligations détectées
- [ ] entités et numéros
- [ ] sources/preuves
- [ ] limites explicites

## Phase 7 — Validation MVP

Tester d'abord sur un corpus de cas représentatifs revus indépendamment :
- produit emballage seul ;
- EEE sans batterie ;
- EEE avec batterie ;
- grande marque internationale ;
- marque locale ;
- private label ;
- produit introuvable ;
- flux transfrontalier ;
- registre indisponible.

Mesurer et publier en interne la précision des états VÉRIFIÉ, le taux de cas vérifiables, les taux NON VÉRIFIÉ / CONTRADICTOIRE, et la fraîcheur des preuves. Définir les seuils avant l'évaluation. L'extension de pays/filières est conditionnée à des résultats acceptables et à des sources auditées dans chaque nouveau périmètre.

## Gate d'intégration Solenda

Le MVP n'est candidat à l'intégration que si :
- les décisions sont explicables ;
- les sources sont auditables ;
- les résultats VÉRIFIÉ sont fiables sur un jeu de référence indépendant et leur couverture est suffisante ;
- NON CONFORME est distingué de NON VÉRIFIÉ ;
- les providers gratuits sont suffisamment stables ;
- le coût opérationnel est acceptable ;
- le moteur est utilisable via API indépendante du dashboard.
