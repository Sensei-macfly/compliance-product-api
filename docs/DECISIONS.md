# Décisions produit — V1

Ce document fixe les décisions déjà validées afin d'éviter les dérives de scope.

## Positionnement

- Le projet est un **MVP autonome de compliance REP/EPR**.
- Il **ne doit pas devenir une plateforme de sourcing**.
- La plateforme de sourcing existe déjà : **Solenda**.
- Si le MVP est concluant, le moteur pourra être intégré à Solenda sous forme d'API/module interne.

## UX : hypothèse à valider

Deux champs avaient été proposés :
1. identifiant produit : EAN / GTIN / UPC / ASIN ;
2. pays où le produit est physiquement stocké chez le fournisseur.

Ils ne suffisent pas, à eux seuls, à déterminer l'entité qui porte l'obligation REP/EPR ni à relier une preuve à un parcours de vente. Ne pas figer l'UX à deux champs avant d'avoir défini le premier parcours et les données nécessaires.

Le pilote devra au minimum pouvoir identifier l'acteur évalué et son rôle dans le parcours, l'entité qui est censée remplir l'obligation, ainsi que les attributs produit nécessaires à la classification. Le pays de stock est un contexte logistique tant que son effet réglementaire dans ce parcours n'a pas été établi.

Ne pas demander systématiquement :
- numéro de TVA ;
- numéro REP personnel ;
- configuration fiscale complexe ;
- compte Amazon.

Demander toutefois un identifiant juridique adéquat lorsque nécessaire pour résoudre une entité. Pour les entreprises non résidentes, la source ADEME peut utiliser un numéro fiscal étranger ou un numéro de TVA intracommunautaire. L'identifiant demandé dépendra de l'entité et de la source du pilote.

## Marchés

Analyse automatique :
- FR
- DE
- IT
- ES
- GB

Le périmètre de faisabilité initial est France / filière EEE. Les cinq marchés et les autres filières sont des extensions conditionnées aux résultats du pilote, pas une exigence de la première livraison.

## Décision

Le moteur conserve des états distincts :
- **VÉRIFIÉ** : preuve suffisante pour le parcours REP/EPR délimité ;
- **NON CONFORME** : obligation applicable et non remplie établies par des preuves suffisantes ;
- **NON VÉRIFIÉ** : données, lien entre acteurs, classification ou source insuffisants ;
- **CONTRADICTOIRE** : preuves pertinentes incompatibles.

La recommandation pratique est « poursuivre » uniquement quand l'état est VÉRIFIÉ ; sinon, « suspendre / vérifier ». Cette recommandation de prudence n'affirme pas que la vente est juridiquement interdite. Pas de score de confiance visible.

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

Les motifs internes comprennent notamment preuve insuffisante, entité ou rôle non résolu, registre indisponible, filière non vérifiée et conflit de données. Ils ne doivent pas être réduits à « NON CONFORME ».

Un résultat positif exige de justifier le lien entre le producteur enregistré, le produit ou la catégorie applicable et le parcours commercial évalué. Un IDU de marque, ou la présence d'un IDU au nom d'une société apparentée, ne suffit pas automatiquement.

## Pilote de faisabilité

- Source candidate vérifiée : jeu ADEME/SYDEREP des producteurs et IDU, filière EEE, France.
- Le jeu comporte une API publique, un téléchargement, une licence ouverte, une actualisation annoncée quotidienne et des champs de filière, identifiant de société, pays, IDU, catégories d'agrément, immatriculation et raison sociale.
- Cette source vérifie un enregistrement d'entité pour une filière/catégorie ; elle ne fournit ni GTIN, ni preuve à elle seule du lien entre l'entité et le vendeur ou le produit analysé.
- Le premier démonstrateur doit donc être décrit comme une vérification de preuves d'enregistrement REP/EPR dans un parcours défini, pas comme une autorisation produit de revente.
- Le rôle du vendeur, son canal de vente et la source du lien vers l'entité responsable restent à préciser et à faire valider avant toute conclusion utilisateur.
- La faisabilité de la source ne vaut pas validation juridique des règles d'applicabilité, ni validation des sources pour l'Allemagne, l'Italie, l'Espagne ou le Royaume-Uni.

## Profil utilisateur

Les prep centers peuvent être configurés en option.

Les numéros de TVA pourront être stockés plus tard, mais ils ne doivent jamais bloquer la V1.

## V1 ≠ avis juridique

L'application produit une **vérification de preuves REP/EPR circonscrite et datée**, pas un avis juridique définitif ni une validation générale de la possibilité de vendre. Toute extension de portée exige des règles documentées, une preuve traçable et une validation métier/réglementaire.
