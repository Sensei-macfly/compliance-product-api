# compliance-product-api

MVP autonome de **vérification de preuves REP/EPR par produit et parcours de vente**, destiné à être testé séparément puis éventuellement intégré à **Solenda**. Il ne produit pas une autorisation générale de vente.

## Objectif

Répondre à la question :

> **Pour ce produit et cette route fournisseur → stock → prep center(s) → Amazon FBA, notre société doit-elle s'enregistrer dans chaque marché cible, ou la couverture REP/EPR d'une entité amont est-elle prouvée ?**

### Parcours à analyser

La société de l'utilisateur vend sur Amazon et utilise FBA. Le modèle doit distinguer :
- EAN / GTIN / UPC / ASIN et attributs nécessaires à la classification ;
- fournisseur et pays où le stock est physiquement détenu ;
- prep centers traversés ;
- programme FBA et pays d'inventaire connus ;
- marché de revente ciblé ;
- entité vendeuse et éventuelle entité amont censée remplir les obligations, avec preuves du lien au produit/SKU.

Le pays d'établissement du fournisseur, le pays de stock et le marché de vente ne sont pas interchangeables. L'API Amazon peut rester facultative ; les faits de route inconnus doivent être fournis ou donner NON VÉRIFIÉ.

### Marchés analysés
- France
- Allemagne
- Italie
- Espagne
- Royaume-Uni

### Sortie

Un résultat par route réelle × marché cible × filière :
- **VÉRIFIÉ**
- **NON CONFORME**
- **NON VÉRIFIÉ**
- **CONTRADICTOIRE**

Le moteur indique aussi si l'enregistrement de notre société manque, est déjà vérifié, si une couverture amont est prouvée, si aucun nouvel enregistrement n'est requis ou si la situation reste indéterminée.
Une action prudente peut être GO/HOLD, mais HOLD ou NON VÉRIFIÉ ne signifie pas que la vente est interdite. **UNKNOWN ≠ COMPLIANT** : aucune preuve insuffisante ne produit VÉRIFIÉ.

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

Les marchés cibles déclarés restent la France, l'Allemagne, l'Italie, l'Espagne et le Royaume-Uni. Les pays fournisseurs peuvent couvrir l'ensemble de l'UE. Une combinaison route/marché/filière non auditée reste NON VÉRIFIÉE.

## Documentation

- [Cahier des charges V1](docs/CDC_V1.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Décisions produit](docs/DECISIONS.md)
- [Sources et APIs](docs/SOURCES_APIS.md)
- [Roadmap MVP](docs/ROADMAP.md)

## Statut

Spécification / recherche technique. Le code applicatif n'est pas encore commencé.
