# compliance-product-api

MVP autonome de **vérification de preuves REP/EPR par produit et parcours de vente**, destiné à être testé séparément puis éventuellement intégré à **Solenda**. Il ne produit pas une autorisation générale de vente.

## Objectif

Répondre à la question :

> **Pour chaque client, quelles routes vers ses marketplaces cibles peuvent être poursuivies sans nouvel enregistrement REP/EPR/IDU, preuves à l'appui ?**

### Parcours à analyser

Le moteur est multi-client et l'analyse doit être générée à partir du profil de chaque client. Pour un vendeur Amazon/FBA, elle distingue :
- EAN / GTIN / UPC / ASIN et attributs nécessaires à la classification ;
- fournisseur et pays où le stock est physiquement détenu ;
- pays où le client détient une immatriculation TVA, comme donnée de profil et non comme preuve REP/EPR ;
- prep centers réellement disponibles ;
- programme FBA et pays d'inventaire permis ou connus ;
- marketplaces cibles demandées ;
- entité vendeuse et éventuelle entité amont censée remplir les obligations, avec preuves du lien au produit/SKU.

Le moteur génère des routes candidates depuis ces faits, sans route unique codée en dur. Le pays d'établissement du fournisseur, le pays de stock et le marché de vente ne sont pas interchangeables. Un pays de TVA ou une marketplace cible ne permet pas d'inférer le pays de stockage FBA. L'API Amazon peut rester facultative ; les faits de route inconnus donnent une décision NON.

Le dashboard et les applications clientes tierces utilisent le même moteur API-first, avec des données isolées par client.

### Marchés analysés
- France
- Allemagne
- Italie
- Espagne
- Royaume-Uni

### Sortie

La décision visible est binaire, par route candidate générée × marché cible × filière :
- **OUI** : les preuves démontrent que la route peut être poursuivie sans nouvel enregistrement de la société évaluée ;
- **NON** : cette démonstration manque ou les preuves établissent qu'une exigence n'est pas satisfaite.

Les états détaillés (VÉRIFIÉ, NON CONFORME, NON VÉRIFIÉ, CONTRADICTOIRE) et les raisons restent internes. Un NON fondé sur des preuves insuffisantes n'est pas une interdiction juridique ni la preuve, à lui seul, que la société évaluée doit s'enregistrer ; l'explication indique ce qui manque. **UNKNOWN ≠ COMPLIANT**.

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
