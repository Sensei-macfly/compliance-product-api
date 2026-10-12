# Roadmap MVP V1

## Objectif

Valider le moteur REP/EPR comme brique indépendante avant intégration éventuelle à Solenda.

## Phase 0 — Spécification

- [x] positionnement : module de compliance, pas plateforme de sourcing
- [x] architecture API-first
- [x] dashboard autonome + Solenda sur le même moteur
- [x] Amazon optionnel
- [x] décision utilisateur OUI/NON uniforme
- [x] états internes conservateurs
- [x] distinction produit / entité / registre / couverture
- [x] profil client basé sur des faits
- [x] suppression de la logique « le client déclare ses pays FBA possibles »
- [x] REP/EPR déjà détenues facultatives mais supportées
- [x] résultat détaillé par route × marché × filière
- [ ] auditer toutes les sources officielles retenues
- [ ] valider les règles pays × filière × rôle
- [ ] constituer un jeu de cas de référence indépendant

## Gate avant développement du moteur

Avant d'autoriser un OUI réel :
- [ ] corpus de référence figé avant évaluation ;
- [ ] 0 faux OUI sur ce corpus pour le périmètre activé ;
- [ ] 100 % des OUI avec chaîne de preuve traçable ;
- [ ] chaîne de preuve documentée ;
- [ ] règle pays × filière × rôle documentée ;
- [ ] données minimales nécessaires définies ;
- [ ] comportement UNVERIFIED défini ;
- [ ] corpus de test revu indépendamment ;
- [ ] cas de données FBA inconnues couvert ;
- [ ] cas de preuve amont incomplète couvert ;
- [ ] cas de registre indisponible couvert ;
- [ ] cas d'inscription propre existante couvert ;
- [ ] cas multi-routes / multi-marchés / multi-filières couvert.

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
- [ ] optional existing seller EPR/REP/IDU registrations

Ne pas demander :
- liste déclarative de pays FBA « possibles »
- routes construites manuellement
- REP/EPR personnelles comme champ obligatoire de l'UX

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

Pilote end-to-end initial :
1. **France / EEE via ADEME/SYDEREP**, pour valider le connecteur de preuve d'enregistrement et la chaîne evidence → règle → verdict, sans considérer l'IDU seul comme preuve de couverture produit/route.

Puis :
2. autres filières France après validation de la chaîne de preuve
3. Allemagne
4. Royaume-Uni
5. Espagne
6. Italie

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
- faits Amazon/FBA explicites si disponibles ;
- règles documentées.

Le moteur :
- [ ] génère les routes candidates raisonnables ;
- [ ] distingue stock fournisseur, prep/transit, stock FBA et marketplace ;
- [ ] ne demande pas au client de construire la route ;
- [ ] n'infère pas un pays FBA uniquement depuis la TVA ;
- [ ] ne produit pas un produit cartésien arbitraire ;
- [ ] marque UNVERIFIED les routes dont une donnée nécessaire manque.

## Phase 7 — Compliance Engine

- [ ] règles pays/filières/rôles
- [ ] upstream coverage
- [ ] existing seller registration assessment
- [ ] local entity resolution
- [ ] résultats par route × marché × filière
- [ ] OUI/NON
- [ ] reason codes
- [ ] missing_context
- [ ] evidence
- [ ] aucune note/score visible
- [ ] NON/UNVERIFIED ne doit pas être présenté comme interdiction ou obligation certaine

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
- [ ] détail par marché et filière
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

## Validation MVP — cas minimaux

- [ ] FBA inconnu
- [ ] fournisseur / entité amont non résolu
- [ ] REP amont présente mais lien produit non prouvé
- [ ] registre indisponible
- [ ] produit multi-filières
- [ ] même GTIN vers plusieurs marchés
- [ ] plusieurs routes vers un même marché
- [ ] couverture amont valide
- [ ] inscription propre déjà valide
- [ ] preuve insuffisante
- [ ] contradictions entre sources

## Gate d'intégration

Intégration possible si :
- décisions explicables ;
- règles auditées ;
- preuves traçables ;
- faux OUI maîtrisés ;
- providers suffisamment stables ;
- API autonome ;
- dashboard et Solenda produisent le même résultat détaillé sur le même payload.
