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

Statut : **V1**.

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
