# compliance-product-api

MVP autonome de **vérification de preuves REP/EPR par produit et parcours de vente**, destiné à être testé séparément puis éventuellement intégré à **Solenda**. Il ne produit pas une autorisation générale de vente.

## Objectif

Répondre à la question :

> **Pour cette route et ce marché, peut-on poursuivre sans nouvel enregistrement REP/EPR/IDU de notre société, preuves à l'appui ?**

### Parcours à analyser

La société de l'utilisateur vend sur Amazon et utilise FBA. Le modèle doit distinguer :
- EAN / GTIN / UPC / ASIN et attributs nécessaires à la classification ;
- fournisseur et pays où le stock est physiquement détenu ;
- prep centers traversés ;
- programme FBA et pays d'inventaire connus ;
- marché de revente ciblé ;
- entité vendeuse et éventuelle entité amont censée remplir les obligations, avec preuves du lien au produit/SKU.

Le pays d'établissement du fournisseur, le pays de stock et le marché de vente ne sont pas interchangeables. L'API Amazon peut rester facultative ; les faits de route inconnus donnent une décision NON.

### Marchés analysés
- France
- Allemagne
- Italie
- Espagne
- Royaume-Uni

### Sortie

La décision visible est binaire, par route réelle × marché cible × filière :
- **OUI** : les preuves démontrent que la route peut être poursuivie sans nouvel enregistrement de notre société ;
- **NON** : cette démonstration manque ou les preuves établissent qu'une exigence n'est pas satisfaite.

Les états détaillés (VÉRIFIÉ, NON CONFORME, NON VÉRIFIÉ, CONTRADICTOIRE) et les raisons restent internes. Un NON fondé sur des preuves insuffisantes n'est pas une interdiction juridique ni la preuve, à lui seul, que notre société doit s'enregistrer ; l'explication indique ce qui manque. **UNKNOWN ≠ COMPLIANT**.

## Positionnement

Ce projet **n'est pas une plateforme de sourcing**. Solenda reste la plateforme de sourcing. Ce repository sert à valider une brique de compliance REP/EPR indépendante, conçue pour devenir ensuite une API/module intégrable.

## Principes V1

- API et registres gratuits/officiels en priorité
- Amazon SP-API facultative
- aucune connexion Amazon/SP-API obligatoire ; les données FBA nécessaires doivent toutefois être saisies/importées
- ne pas demander de numéro de TVA sans besoin de résolution identifié
- pas d'API commerciale payante indispensable
- aucun score visible
- toutes les conclusions doivent être explicables et auditables

Les marchés cibles déclarés restent la France, l'Allemagne, l'Italie, l'Espagne et le Royaume-Uni. Les pays fournisseurs peuvent couvrir l'ensemble de l'UE. Une combinaison route/marché/filière non auditée reçoit NON.

## Documentation

- [Cahier des charges V1](docs/CDC_V1.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Décisions produit](docs/DECISIONS.md)
- [Sources et APIs](docs/SOURCES_APIS.md)
- [Roadmap MVP](docs/ROADMAP.md)

## Statut

Spécification / recherche technique. Le code applicatif n'est pas encore commencé.
