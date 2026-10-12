# compliance-product-api

MVP autonome de **vérification REP/EPR par produit et parcours de vente**, conçu pour être testé séparément puis intégré à **Solenda** via API.

## Objectif

Répondre à la question :

> **À partir des faits connus sur le produit, le sourcing, la marketplace cible et la configuration du client, quelles routes permettent de poursuivre la vente sans nouvel enregistrement REP/EPR/IDU ?**

Le moteur doit **déduire les routes candidates**. Il ne doit pas demander au client de deviner lui-même les pays FBA possibles ni de construire manuellement le trajet.

## Deux modes d'utilisation

### Dashboard autonome
L'utilisateur saisit les données minimales nécessaires, typiquement :
- identifiant produit : EAN / GTIN / UPC / ASIN ;
- pays de sourcing / pays réel du stock fournisseur ;
- marketplace cible.

Le profil client fournit les faits persistants :
- entité vendeuse ;
- pays d'établissement ;
- pays d'immatriculation TVA ;
- prep centers réellement disponibles ;
- paramètres Amazon connus si disponibles.

### Solenda via API
Solenda peut transmettre directement les données déjà connues :
- EAN / GTIN / ASIN ;
- données produit ;
- pays de sourcing / stock fournisseur ;
- marketplace cible ;
- fournisseur ;
- contexte compte client ;
- données Amazon/FBA déjà disponibles.

Le dashboard et Solenda doivent utiliser **exactement le même moteur métier et le même contrat API**.

## Principes V1

- API-first / headless-first
- aucune connexion Amazon obligatoire
- Amazon SP-API uniquement comme enrichissement optionnel
- APIs et registres gratuits/officiels en priorité
- aucune API commerciale payante indispensable
- aucun score visible
- décision utilisateur : **OUI / NON**
- états techniques internes conservés pour expliquer le verdict
- **UNKNOWN ≠ COMPLIANT**
- un numéro REP/IDU trouvé ne prouve jamais, à lui seul, la couverture d'un produit ou d'un flux

## Positionnement

Ce projet **n'est pas une plateforme de sourcing**. Solenda reste la plateforme de sourcing. Ce repository sert à valider une brique de compliance réutilisable sous forme d'API.

## Documentation

- [Cahier des charges V1](docs/CDC_V1.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Décisions produit](docs/DECISIONS.md)
- [Sources et APIs](docs/SOURCES_APIS.md)
- [Roadmap MVP](docs/ROADMAP.md)

## Statut

Spécification / recherche technique. Le code applicatif n'est pas encore commencé.
