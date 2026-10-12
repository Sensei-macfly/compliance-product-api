# Roadmap MVP V1

## Objectif

Valider un moteur de couverture REP/EPR amont par produit et marché avant intégration à Solenda.

## Phase 0 — Spécification

- [x] API-first
- [x] dashboard + Solenda sur le même moteur
- [x] OUI/NON uniforme
- [x] UNKNOWN ≠ COMPLIANT
- [x] identifiant + stock fournisseur physique + marché cible + product_handling_mode explicite
- [x] suppression des pays FBA devinés
- [x] suppression du Route Generator V1
- [x] suppression prep center / transit / stockage FBA du noyau V1
- [x] bundles / repackaging repoussés en V2
- [x] OUI applicable = couverture amont vérifiée
- [x] immatriculation propre vendeur exclue comme chemin favorable V1
- [x] UX : numéros visibles même sur NON
- [x] messages utilisateur français / codes techniques anglais
- [x] fail-closed si un fait réglementaire requis est absent ou hors périmètre V1
- [x] UNKNOWN/absence de product_handling_mode interdit OUI
- [x] APPLICABLE + OUI exige evidence[] non vide
- [ ] auditer les sources officielles
- [ ] valider les règles pays × filière × rôle
- [ ] constituer le corpus de référence

## Gate avant activation des vrais OUI

Pour chaque combinaison pays × filière × rôle :
- [ ] corpus figé avant évaluation
- [ ] 0 faux OUI
- [ ] 100 % des OUI traçables
- [ ] aucune filière APPLICABLE avec OUI et evidence=[]
- [ ] au moins 3 cas positifs indépendamment revus
- [ ] au moins 3 cas négatifs/ambigus
- [ ] 100 % des cas positifs reconnus
- [ ] source indisponible => NON/UNVERIFIED
- [ ] couverture amont incomplète => NON/UNVERIFIED
- [ ] applicabilité filière non résolue => NON
- [ ] fait réglementaire requis absent/hors scope => NON/UNVERIFIED
- [ ] aucune hypothèse logistique favorable inventée pour produire OUI
- [ ] violation connue de l'hypothèse produit V1 => NON/UNVERIFIED

## Phase 1 — Fondations

- [ ] Next.js / TypeScript
- [ ] PostgreSQL / Prisma
- [ ] auth
- [ ] isolation tenant
- [ ] POST /api/v1/analyses
- [ ] evidence model
- [ ] provider abstraction
- [ ] identifier validation
- [ ] logs / observabilité

## Phase 2 — Profil client minimal

- [ ] seller entity
- [ ] country of establishment
- [ ] VAT registration countries optionnels

Ne pas demander :
- prep centers
- pays FBA
- route construite
- immatriculations REP propres comme chemin positif V1

## Phase 3 — Product & Entity Resolution

- [ ] GTIN validation
- [ ] GS1 adapter si exploitable
- [ ] GLEIF
- [ ] EUIPO
- [ ] France Entreprises / RNE
- [ ] Companies House
- [ ] entity normalization
- [ ] corporate graph minimal

Amazon SP-API : optionnelle pour l'enrichissement produit.

## Phase 4 — Classification REP

- [ ] PACKAGING
- [ ] EEE/WEEE
- [ ] BATTERY
- [ ] applicability
- [ ] stream_scope_state
- [ ] tests

## Phase 5 — Registres

Pilote initial :
1. **France / EEE via ADEME/SYDEREP** pour valider la chaîne registre → evidence → coverage assessment, sans considérer l'IDU seul comme preuve de couverture produit.

Puis :
2. autres filières France
3. Allemagne
4. Royaume-Uni
5. Espagne
6. Italie

Pour chaque source :
- [ ] source officielle
- [ ] ToS / automatisation
- [ ] normalisation
- [ ] cache / sync
- [ ] evidence
- [ ] tests

## Phase 6 — Upstream Coverage Engine

- [ ] règles marché × filière × rôle
- [ ] identification entité amont pertinente
- [ ] registry matching
- [ ] distinction registration verified / product coverage verified
- [ ] UPSTREAM_VERIFIED
- [ ] UPSTREAM_UNPROVEN
- [ ] résultats marché × filière
- [ ] OUI/NON
- [ ] reason codes
- [ ] user_message_fr
- [ ] evidence
- [ ] aucune note visible

## Phase 7 — Dashboard autonome

Entrées :
- [ ] identifier
- [ ] supplier_stock_country
- [ ] target_marketplace
- [ ] product_handling_mode via confirmation explicite non cochée par défaut

Afficher immédiatement :
- [ ] OUI/NON
- [ ] numéros officiels trouvés
- [ ] titulaire / registre / pays
- [ ] couverture produit vérifiée ou non
- [ ] éléments validés
- [ ] cause du NON ou point à vérifier
- [ ] détail par filière
- [ ] preuves
- [ ] limites

## Phase 8 — Intégration Solenda

- [ ] même endpoint
- [ ] payload enrichi facultatif
- [ ] données produit/sourcing/marketplace réutilisées
- [ ] aucun accès direct à la DB Solenda
- [ ] versioning API stable

## V2 — explicitement hors V1

- prep centers
- transit à impact réglementaire
- bundle
- repackaging
- modification d'emballage
- règles dépendantes d'un pays de stockage FBA réellement connu
- analyses post-envoi Amazon

## Validation MVP — cas minimaux

- [ ] produit non résolu
- [ ] applicabilité filière incertaine
- [ ] entité amont non résolue
- [ ] numéro officiel non trouvé
- [ ] numéro trouvé mais couverture produit non démontrée
- [ ] registre indisponible
- [ ] produit multi-filières
- [ ] couverture amont complète
- [ ] contradiction entre sources
- [ ] règle nécessitant un fait réglementaire hors scope/non disponible => NON
- [ ] product_handling_mode absent => NON/UNVERIFIED
- [ ] product_handling_mode UNKNOWN => NON/UNVERIFIED
- [ ] product_handling_mode BUNDLED => NON/UNVERIFIED
- [ ] product_handling_mode REPACKAGED => NON/UNVERIFIED
- [ ] ORIGINAL_UNMODIFIED + preuves complètes => OUI possible
- [ ] APPLICABLE + OUI + evidence=[] => réponse invalide / NON-UNVERIFIED
- [ ] produit connu comme bundlé/reconditionné/modifié => NON/UNVERIFIED
- [ ] même GTIN analysé sur plusieurs marchés via appels distincts

## Gate d'intégration

Intégration si :
- décisions explicables ;
- règles auditées ;
- preuves traçables ;
- faux OUI maîtrisés ;
- providers suffisamment stables ;
- API autonome ;
- dashboard et Solenda produisent le même résultat sur le même payload.
