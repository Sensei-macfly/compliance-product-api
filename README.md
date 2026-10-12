# compliance-product-api

MVP autonome de **pré-validation REP/EPR par produit**, destiné à être testé séparément puis éventuellement intégré à **Solenda**.

## Objectif

Répondre à la question :

> **Ce produit est stocké chez mon fournisseur dans ce pays. Dans quels marchés puis-je le revendre avec mon modèle actuel, sans nouvelle immatriculation REP/EPR/IDU ?**

### Entrées V1
1. EAN / GTIN / UPC / ASIN
2. Pays où le stock fournisseur est physiquement situé

### Marchés analysés
- France
- Allemagne
- Italie
- Espagne
- Royaume-Uni

### Sortie
Décision binaire par pays :
- **OUI**
- **NON**

Règle fondamentale : **UNKNOWN ≠ COMPLIANT**. Une preuve insuffisante ne doit jamais produire un OUI.

## Positionnement

Ce projet **n'est pas une plateforme de sourcing**. Solenda reste la plateforme de sourcing. Ce repository sert à valider une brique de compliance REP/EPR indépendante, conçue pour devenir ensuite une API/module intégrable.

## Principes V1

- API et registres gratuits/officiels en priorité
- Amazon SP-API facultative
- aucun compte Amazon Professional requis pour utiliser la V1
- aucun numéro de TVA obligatoire
- pas d'API commerciale payante indispensable
- aucun score visible : décision OUI/NON
- toutes les conclusions doivent être explicables et auditables

## Documentation

- [Cahier des charges V1](docs/CDC_V1.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Décisions produit](docs/DECISIONS.md)
- [Sources et APIs](docs/SOURCES_APIS.md)
- [Roadmap MVP](docs/ROADMAP.md)

## Statut

Spécification / recherche technique. Le code applicatif n'est pas encore commencé.
