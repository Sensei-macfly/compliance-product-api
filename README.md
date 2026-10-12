# compliance-product-api

MVP autonome de **pré-validation REP/EPR par produit et marché**, conçu pour être testé séparément puis intégré à **Solenda** via API.

## Objectif

Répondre à la question :

> **Pour ce produit et ce marché cible, peut-on démontrer qu'une couverture REP/EPR amont suffisante existe, de sorte que l'entité vendeuse n'ait pas à porter elle-même une nouvelle obligation REP/EPR pour les filières applicables ?**

Le moteur travaille à partir de faits simples :
- identifiant produit : EAN / GTIN / UPC / ASIN ;
- pays où le produit est physiquement stocké chez le fournisseur au moment du sourcing ;
- marketplace cible ;
- identité de l'entité vendeuse depuis le profil client.

## Simplification V1

La V1 **ne modélise pas la route logistique**.

Hors noyau V1 :
- prep centers ;
- transit ;
- pays de stockage FBA ;
- prédiction de ce qu'Amazon fera du stock ;
- bundles ;
- reconditionnement / modification d'emballage.

Ces sujets pourront être traités dans une V2 si une règle réglementaire ou un cas métier le nécessite.

## Principes V1

- API-first / headless-first
- dashboard autonome + Solenda sur le même moteur
- Amazon SP-API facultative et limitée à l'enrichissement produit
- sources officielles/gratuites prioritaires
- aucun score visible
- décision utilisateur : **OUI / NON**
- **UNKNOWN ≠ COMPLIANT**
- un numéro REP/IDU trouvé ne prouve jamais, à lui seul, la couverture du produit
- un OUI sur une filière applicable exige une **couverture amont vérifiée**
- une immatriculation propre du vendeur n'est **pas** considérée comme le scénario favorable recherché par la V1

## Positionnement

Ce projet **n'est pas une plateforme de sourcing** et **n'est pas un moteur logistique**. Solenda reste la plateforme de sourcing.

## Documentation

- [Cahier des charges V1](docs/CDC_V1.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Décisions produit](docs/DECISIONS.md)
- [UX V1](docs/UX_V1.md)
- [Sources et APIs](docs/SOURCES_APIS.md)
- [Roadmap MVP](docs/ROADMAP.md)
- [Collaboration & source de vérité](docs/COLLABORATION.md)

## Statut

Spécification / recherche technique. Le code applicatif n'est pas encore commencé.
