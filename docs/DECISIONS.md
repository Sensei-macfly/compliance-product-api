# Décisions produit — V1

Ce document fixe les décisions déjà validées afin d'éviter les dérives de scope.

## Positionnement

- Le projet est un **MVP autonome de compliance REP/EPR**.
- Il **ne doit pas devenir une plateforme de sourcing**.
- La plateforme de sourcing existe déjà : **Solenda**.
- Le moteur est **API-first** : le dashboard du projet et les applications clientes tierces appellent le même contrat, avec les données isolées par client/tenant.
- Le MVP doit fonctionner pour plusieurs clients, pas seulement pour la société du créateur.
- Si le MVP est concluant, le moteur pourra aussi être intégré à Solenda sous forme d'API/module.

## UX : hypothèse à valider

Deux champs avaient été proposés :
1. identifiant produit : EAN / GTIN / UPC / ASIN ;
2. pays où le produit est physiquement stocké chez le fournisseur.

Ils ne suffisent pas, à eux seuls, à déterminer l'entité qui porte l'obligation REP/EPR ni à générer les routes possibles. Ne pas figer l'UX à deux champs ni coder une route unique ; définir plutôt les données de profil client et les transitions nécessaires au générateur.

Le pilote devra au minimum pouvoir identifier l'acteur évalué et son rôle dans le parcours, l'entité qui est censée remplir l'obligation, ainsi que les attributs produit nécessaires à la classification. Le pays de stock est un contexte logistique tant que son effet réglementaire dans ce parcours n'a pas été établi.

Modèle commercial de départ : le créateur vend sur Amazon et s'approvisionne dans plusieurs pays européens, mais le produit doit évaluer l'entité vendeuse de chaque client. Les marchandises peuvent passer par des prep centers avant Amazon FBA, puis être vendues sur plusieurs marketplaces.

Générer les routes candidates depuis le profil client, les pays de sourcing/stock, le produit, les prep centers disponibles, la configuration FBA et les marketplaces demandées. Distinguer établissement fournisseur, stock physique, prep centers, pays FBA, programme et cible. Ne pas coder un trajet prédéfini ni croiser arbitrairement des pays sans données de transition.

Les pays d'immatriculation TVA du client peuvent être un attribut de configuration pour filtrer des routes selon des règles documentées ; ils ne prouvent ni éligibilité FBA ni conformité/couverture REP/EPR.

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

Marchés cibles déclarés pour la V1 : FR, DE, IT, ES, GB. Les pays fournisseurs/stock peuvent inclure d'autres pays de l'UE ; générer un résultat pour chaque route candidate et marché demandé. Toute combinaison sans règle/source auditée reçoit NON dans l'interface et conserve NON VÉRIFIÉ en interne. La matrice des marchés et routes n'est pas encore validée pays par pays.

## Décision

Le moteur conserve des états techniques distincts en interne :
- **VÉRIFIÉ** : preuve suffisante pour le parcours REP/EPR délimité ;
- **NON CONFORME** : obligation applicable et non remplie établies par des preuves suffisantes ;
- **NON VÉRIFIÉ** : données, lien entre acteurs, classification ou source insuffisants ;
- **CONTRADICTOIRE** : preuves pertinentes incompatibles.

La sortie utilisateur reste strictement binaire : **OUI** seulement quand l'état interne est VÉRIFIÉ et que l'évaluation d'inscription est favorablement établie ; **NON** dans tous les autres cas. Les états internes ne doivent pas apparaître comme une troisième décision commerciale. Un NON pour données insuffisantes n'affirme ni que la vente est interdite ni que l'entité vendeuse évaluée est certainement tenue de s'enregistrer ; une explication indique les preuves manquantes. Pas de score de confiance visible.

En plus de l'état des preuves, chaque route doit répondre à la question métier via un résultat distinct : `SELLER_REGISTRATION_MISSING`, `SELLER_REGISTRATION_VERIFIED`, `UPSTREAM_COVERAGE_VERIFIED`, `NO_SELLER_REGISTRATION_REQUIRED` ou `UNRESOLVED`. Ne pas traduire `UNRESOLVED` en « l'entité vendeuse doit s'enregistrer » sans règle qui établit sa responsabilité.

## Amazon

- Le cas d'usage initial est celui de la société du créateur, vendeuse sur Amazon ; le moteur évalue toutefois l'entité vendeuse de chaque client. Amazon intervient comme marketplace et fournisseur du fulfillment FBA selon le programme utilisé.
- Le moteur doit connaître le programme de fulfillment et les pays d'inventaire FBA pertinents pour chaque route. Il ne doit pas déduire ces pays du pays fournisseur ou du marché Amazon ciblé.
- Amazon SP-API est **facultative**.
- Le moteur doit accepter des données de route saisies/importées sans identifiants SP-API ; si les pays de stock FBA nécessaires sont inconnus, afficher NON et conserver NON VÉRIFIÉ en interne.
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
2. l'entité vendeuse du client et son rôle identifiés ;
3. fournisseur, fabricant/producteur et entité amont identifiés ;
4. enregistrement vérifié dans le pays et la filière/catégorie applicables ;
5. preuve reliant l'entité enregistrée au produit/SKU et au parcours ;
6. applicabilité à l'entité vendeuse évaluée, au marché cible et au parcours FBA analysée selon une règle vérifiée.

## Données insuffisantes

Les motifs internes comprennent notamment preuve insuffisante, entité ou rôle non résolu, registre indisponible, filière non vérifiée et conflit de données. Ils ne doivent pas être réduits à « NON CONFORME ».

Un résultat positif exige de justifier le lien entre le producteur enregistré, le produit ou la catégorie applicable et le parcours commercial évalué. Un IDU de marque, ou la présence d'un IDU au nom d'une société apparentée, ne suffit pas automatiquement.

## Faisabilité de la source et pilote end-to-end

- Source candidate vérifiée : jeu ADEME/SYDEREP des producteurs et IDU, filière EEE, France. Il vérifie uniquement la disponibilité d'une preuve d'enregistrement française.
- Le jeu comporte une API publique, un téléchargement, une licence ouverte, une actualisation annoncée quotidienne et des champs de filière, identifiant de société, pays, IDU, catégories d'agrément, immatriculation et raison sociale.
- Cette source vérifie un enregistrement d'entité pour une filière/catégorie ; elle ne fournit ni GTIN, ni preuve à elle seule du lien entre l'entité et la société vendeuse évaluée, le fournisseur, le produit ou le trajet.
- Le pilote end-to-end doit valider la génération des routes à partir de profils et contraintes clients ; les cas de test ne doivent pas devenir des routes fixes codées en dur.
- Un identifiant amont ne signifie pas automatiquement que la société vendeuse évaluée n'a rien à enregistrer : l'applicabilité et les preuves de couverture doivent être vérifiées par marché, rôle, filière et catégorie.
- La faisabilité de la source ne vaut pas validation juridique des règles d'applicabilité, ni validation des sources pour l'Allemagne, l'Italie, l'Espagne ou le Royaume-Uni.

## Profil utilisateur

Pour chaque client, les fournisseurs/sources, pays d'immatriculation TVA, prep centers disponibles, programmes/pays FBA et marchés cibles doivent être configurables. Les applications clientes peuvent envoyer ces données enrichies via la même API que le dashboard.

Ne pas collecter un numéro de TVA sans besoin précis. Si la résolution sûre d'une entité étrangère exige un identifiant fiscal ou un numéro de TVA que les données disponibles ne fournissent pas, l'analyse concernée reste NON VÉRIFIÉE.

## V1 ≠ avis juridique

L'application produit une **vérification de preuves REP/EPR circonscrite et datée**, pas un avis juridique définitif ni une validation générale de la possibilité de vendre. Toute extension de portée exige des règles documentées, une preuve traçable et une validation métier/réglementaire.
