# Décisions produit — V1

Ce document fixe les décisions déjà validées afin d'éviter les dérives de scope.

## Positionnement

- Le projet est un **MVP autonome de compliance REP/EPR**.
- Il **ne doit pas devenir une plateforme de sourcing**.
- La plateforme de sourcing existe déjà : **Solenda**.
- Si le MVP est concluant, le moteur pourra être intégré à Solenda sous forme d'API/module interne.

## UX V1

Deux champs obligatoires uniquement :
1. identifiant produit : EAN / GTIN / UPC / ASIN ;
2. pays où le produit est physiquement stocké chez le fournisseur.

L'utilisateur ne doit pas avoir à saisir :
- numéro de TVA ;
- numéro REP personnel ;
- configuration fiscale complexe ;
- compte Amazon.

## Marchés

Analyse automatique :
- FR
- DE
- IT
- ES
- GB

## Décision

Le verdict utilisateur est **binaire** :
- OUI
- NON

Aucun score de confiance n'est affiché comme décision.

Règle : **si le moteur ne peut pas établir suffisamment le OUI, il retourne NON**.

## Amazon

- Amazon SP-API est **facultative**.
- La V1 doit fonctionner sans compte vendeur Amazon Professional.
- Une connexion Amazon pourra améliorer :
  - ASIN ↔ EAN/GTIN ;
  - titre ;
  - marque ;
  - manufacturer ;
  - product type ;
  - catégories et attributs.
- Amazon n'est jamais une preuve juridique de conformité REP.

## Coûts / providers

La V1 doit fonctionner sans dépendance commerciale obligatoire.

Ne pas rendre obligatoires :
- OpenCorporates payant ;
- Orbis ;
- Dun & Bradstreet ;
- Creditsafe ;
- Kompass ;
- autres APIs payantes.

## Responsabilité REP

Ne jamais considérer que :

```text
numéro REP trouvé = produit/lot couvert
```

Le moteur doit distinguer :
1. produit identifié ;
2. marque / titulaire / fabricant identifiés ;
3. entité juridique locale identifiée ;
4. enregistrement REP vérifié ;
5. applicabilité au flux analysé.

## Données insuffisantes

Les statuts internes peuvent comprendre :
- preuve insuffisante ;
- entité non résolue ;
- registre indisponible ;
- filière non vérifiée ;
- conflit de données.

Mais le verdict commercial visible reste OUI/NON.

## Profil utilisateur

Les prep centers peuvent être configurés en option.

Les numéros de TVA pourront être stockés plus tard, mais ils ne doivent jamais bloquer la V1.

## V1 ≠ avis juridique

L'application produit une **pré-validation automatisée conservatrice**, pas un avis juridique définitif.
