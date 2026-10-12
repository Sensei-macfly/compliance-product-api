# Roadmap MVP V1

## Objectif

Valider séparément la faisabilité technique et réglementaire du moteur REP/EPR avant toute intégration à Solenda.

## Phase 0 — Spécification

- [x] positionnement MVP autonome
- [x] confirmer le modèle de départ : le créateur vend sur Amazon et utilise FBA ; l'outil doit évaluer plusieurs clients
- [x] distinguer fournisseur, stockage initial, prep center, stock FBA et marché cible
- [ ] spécifier la génération dynamique des routes à partir du profil client, des sources, du produit, des prep centers, de FBA et des marketplaces demandées
- [ ] définir les sources des contraintes opérationnelles FBA et des transitions logistiques ; ne pas les déduire des pays de TVA
- [ ] définir les données d'entité, preuves et contraintes transmises par le client ou une application tierce
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

- [ ] valider le générateur sur des jeux de cas multi-client représentant différentes sources, prep centers, profils FBA et marketplaces, sans route codée en dur ;
- [ ] faire valider par une personne compétente les règles REP/EPR des marchés et rôles effectivement pris en charge ;
- [ ] produire une chaîne de preuves de bout en bout ; l'IDU ADEME seul ne relie pas une société au produit/GTIN ni à l'entité vendeuse du client ;
- [ ] décrire la stratégie de synchronisation, l'attribution de la source, la fraîcheur et les cas d'indisponibilité ;
- [ ] définir les mesures de fiabilité positive, de couverture et de résultats indéterminés sur des cas de référence tenus à l'écart des règles.

Si cette gate échoue, ne jamais produire OUI à partir d'une route ou d'une preuve supposée ; retourner NON avec une explication, sans prétendre qu'une obligation légale ou une interdiction est établie.

## Phase 1 — Fondations

- [ ] créer un générateur de routes multi-étapes depuis les options de sourcing/stock, le profil client, les prep centers disponibles, les contraintes FBA et les marketplaces demandées ;
- [ ] produire un résultat par route générée × marché cible × filière ; afficher NON pour toute combinaison dont les règles ou sources ne sont pas auditées, tout en conservant NON VÉRIFIÉ en interne ;
- [ ] commencer la vérification de source par l'ADEME France / EEE, sans présenter ce connecteur comme un parcours end-to-end validé ;
- [ ] Next.js / TypeScript
- [ ] PostgreSQL / Prisma
- [ ] schéma de données
- [ ] auth simple
- [ ] isolation tenant pour profils, routes, analyses et preuves
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

Ordre recommandé pour les sources :
1. France / EEE (connecteur de preuve d'enregistrement)
2. autres filières France après validation de la chaîne de preuves
3. Allemagne
4. Royaume-Uni
5. Espagne
6. Italie

L'analyse juridique et les tests doivent être définis par marché cible × filière × rôle du vendeur, avec les pays fournisseurs/stock et le parcours FBA comme contexte. La couverture d'un seul registre ne permet pas d'affirmer que les autres combinaisons sont vérifiées.

Pour chaque pays :
- [ ] provider officiel
- [ ] normalisation
- [ ] cache local
- [ ] evidence
- [ ] tests
- [ ] reason codes

## Phase 5 — Compliance Engine

- [ ] règles pays/filières
- [ ] conserver VÉRIFIÉ / NON CONFORME / NON VÉRIFIÉ / CONTRADICTOIRE comme états internes
- [ ] afficher seulement OUI / NON et expliquer pourquoi un NON peut être dû à une preuve manquante, sans conclure à une interdiction
- [ ] décision d'inscription : requise/absente, société déjà enregistrée, couverture amont vérifiée, non requise, ou indéterminée
- [ ] action HOLD distincte d'une conclusion sur le droit général de vendre
- [ ] reason codes
- [ ] comportement fail-safe
- [ ] aucun score visible

## Phase 6 — Dashboard de test

- [ ] champ identifiant
- [ ] fournisseur et pays d'établissement
- [ ] pays de stock initial et pays des prep centers
- [ ] programme FBA et pays d'inventaire connus
- [ ] fiche produit
- [ ] matrice de toutes les routes candidates générées × marchés cibles demandés
- [ ] routes/règles non prises en charge affichées NON, avec explication claire et état technique conservé en interne
- [ ] configuration multi-client : pays TVA, prep centers disponibles, transitions logistiques, programmes et pays FBA permis/connus
- [ ] API directe pour que des applications tierces envoient des données enrichies au même moteur
- [ ] raisons et preuves associées à chaque décision OUI / NON
- [ ] entité qui devrait s'enregistrer et preuves de couverture amont
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
- itinéraires passant par un prep center dans un pays tiers ;
- mêmes fournisseur/GTIN vers deux marchés cibles différents ;
- même marché cible avec des pays de stock initial/FBA différents ;
- routes avec couverture amont liée au SKU et routes sans preuve de ce lien ;
- obligations établies de l'entité vendeuse évaluée, de l'entité amont et cas où le rôle reste indéterminé ;
- pays de stock FBA inconnu ;
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
