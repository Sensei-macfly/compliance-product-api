# Sources & APIs — V1

Objectif : privilégier les sources **officielles, gratuites et légalement automatisables**. Toute source doit être encapsulée dans un provider remplaçable.

## Résolution produit

### GS1 / Verified by GS1
Usage :
- validation GTIN ;
- titulaire/licencié GTIN ;
- marque ;
- informations produit disponibles.

Contraintes :
- accès public limité ;
- ne pas dépendre du mode gratuit pour des volumes importants ;
- ne jamais utiliser GS1 comme preuve REP.

Statut : **V1 si accès exploitable / adapter obligatoire**.

### Amazon SP-API
Usage :
- ASIN ↔ EAN/GTIN ;
- marque ;
- manufacturer ;
- product type ;
- catégorie ;
- attributs catalogue.

Contraintes :
- connexion Amazon requise ;
- ne doit jamais être obligatoire ;
- le MVP doit fonctionner sans compte Amazon Professional.

Statut : **OPTIONNEL / ENRICHISSEMENT**.

## Résolution corporate

### GLEIF API
Usage :
- LEI ;
- identité légale ;
- direct parent ;
- ultimate parent ;
- relations corporate ;
- fuzzy search selon capacités disponibles.

Statut : **V1**.

### EUIPO API
Usage :
- titulaire/propriétaire de marque ;
- résolution marque → entité.

Statut : **V1**.

### France — API Recherche Entreprises / INPI-RNE
Usage :
- identité légale ;
- SIREN/SIRET ;
- établissements ;
- activité ;
- raison sociale.

Statut : **V1**.

### Royaume-Uni — Companies House API
Usage :
- company number ;
- nom légal ;
- statut ;
- adresse.

Statut : **V1**.

### OpenCorporates
Usage futur :
- fallback international de résolution d'entités.

Contrainte :
- API commerciale pour usage sérieux.

Statut : **PAS OBLIGATOIRE V1 / V1.5 éventuelle**.

## Registres REP

### France
Sources officielles ADEME / registres producteurs disponibles.

Chercher :
- IDU ;
- filière ;
- entité ;
- statut ;
- date de vérification.

Statut : **SOURCE CANDIDATE DU PILOTE FRANCE / EEE**. Ce registre permet de vérifier qu'une entité est déclarée pour une filière et des catégories. Il ne contient pas de GTIN/ASIN et ne prouve pas, seul, que cette entité remplit l'obligation pour le vendeur et le produit du parcours analysé.

#### Source vérifiée le 12 octobre 2026

- Jeu officiel ADEME/SYDEREP : [REP - Liste des producteurs enregistrés dans SYDEREP avec leur IDU](https://data.ademe.fr/datasets/rep-producteurs-idu)
- API publique : `https://data.ademe.fr/data-fair/api/v1/datasets/rep-producteurs-idu/`
- API des lignes filtrables et paginées : `GET /lines` ; filtres disponibles par colonne, téléchargement complet via `/full`.
- Le jeu annonce une actualisation quotidienne et une licence ouverte ; environ 309 000 lignes au moment de la vérification.
- Champs utiles observés : `Filiere`, `Identifiant_societe`, `Pays`, `Identifiant_unique`, `Categories_agrement`, `Immatriculation`, `Raison_Sociale`.
- `Identifiant_societe` peut désigner un SIREN/SIRET pour une entreprise résidente ou un identifiant fiscal / numéro de TVA pour une entreprise étrangère. Ne pas imposer un format d'identifiant universel.
- La réponse testée sur `Filiere=EEE` renvoie des lignes d'entités étrangères possédant un IDU français. Ne pas traiter le champ `Pays` comme le pays du marché cible ; vérifier son sens dans le référentiel ADEME avant de l'utiliser.
- L'API limite les appels anonymes à 600 requêtes par tranche de 60 secondes et limite aussi le débit et le temps de traitement. Préférer un téléchargement et une synchronisation locale quotidienne au trafic de requêtes par analyse.
- La page ADEME signale que la suppression d'un IDU collectif dépend de l'initiative de l'éco-organisme et que la liste ne reflète pas les producteurs ayant effectué une déclaration de données. La date de mise à jour quotidienne n'est donc pas, à elle seule, une garantie que chaque fait réglementaire lié au producteur est complet.
- Les codes de filière et catégories doivent être chargés depuis le référentiel ADEME fourni et versionnés. Le schéma et les listes de codes peuvent évoluer.

Limite centrale : le jeu établit un enregistrement d'une entité dans une filière/catégorie. Il ne prouve pas la relation entre le produit identifié par GTIN, le vendeur évalué, son fournisseur et l'entité enregistrée. Cette relation doit venir d'une source indépendante ou rester **NON VÉRIFIÉE**.

Pour un client vendeur sur Amazon et utilisant FBA, l'API ADEME peut contribuer à vérifier une inscription française pour l'entité et la filière interrogées. Elle ne détermine pas si la société vendeuse évaluée doit elle-même s'enregistrer, ne prouve pas qu'un fournisseur couvre ce GTIN/SKU, et ne décrit ni les passages en prep center ni les pays de stockage FBA. Ces faits et l'applicabilité doivent être évalués séparément pour chaque route candidate et marché cible.

Références officielles :
- [Recherche de producteurs REP (ADEME)](https://filieres-rep.ademe.fr/recherche-producteurs)
- [Comprendre l'IDU et les rôles vendeur/producteur (ADEME)](https://filieres-rep.ademe.fr/identifiant-unique)
- [Documentation de l'API publique](https://data.ademe.fr/data-fair/api/v1/datasets/rep-producteurs-idu/api-docs.json)
- Licence indiquée sur le jeu : [Licence Ouverte / Open Licence](https://www.etalab.gouv.fr/licence-ouverte-open-licence). Respecter ses obligations et les limites d'usage de l'API.

Cette vérification de source ne valide pas les règles juridiques françaises applicables à chaque rôle, canal ou produit ; elle ne valide pas non plus les fournisseurs de données des quatre autres marchés.

### Allemagne
Packaging :
- LUCID / ZSVR ;
- privilégier accès officiel / bulk XML / token lorsque disponible.

EEE / batteries :
- sources officielles Stiftung EAR et registres associés.

Statut : **V1**.

### Royaume-Uni
- GOV.UK EPR packaging ;
- registres WEEE ;
- registres batteries si applicables ;
- Companies House pour les jointures d'entité.

Préférer CSV/ODS/XLSX structurés et synchronisation locale.

Statut : **V1**.

### Espagne
Créer des adapters pour les registres officiels :
- packaging ;
- EEE/WEEE ;
- batteries.

Si les données officielles ne permettent pas une preuve exploitable :
- verdict conservateur NON.

Statut : **V1, après FR/DE/GB**.

### Italie
Créer des adapters pour :
- Registro AEE ;
- batteries ;
- emballages / mécanismes officiels disponibles.

Registro Imprese commercial ne doit pas être une dépendance V1.

Statut : **V1, après FR/DE/GB**.

## Providers commerciaux exclus de la dépendance V1

Ne pas rendre obligatoires :
- OpenCorporates commercial ;
- Orbis / Bureau van Dijk ;
- Dun & Bradstreet ;
- Creditsafe ;
- Kompass.

Ils pourront servir ultérieurement de fallback si le modèle économique le justifie.

## GitHub / open source

Lorsqu'une bibliothèque open source est retenue :
- licence compatible SaaS commercial ;
- activité/maintenance récente ;
- pas de scraping contraire aux ToS ;
- dépendances raisonnables ;
- tests présents ;
- code auditable ;
- possibilité de la remplacer.

Les libs open source doivent accélérer le développement, pas devenir une source de vérité réglementaire.
