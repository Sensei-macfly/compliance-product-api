# Décisions produit — V1

Ce document fixe les décisions validées afin d'éviter les dérives de scope.

## Positionnement

- Le projet est un **MVP autonome de compliance REP/EPR**.
- Il **ne doit pas devenir une plateforme de sourcing**.
- Solenda reste la plateforme de sourcing.
- Le moteur est **API-first**.
- Le dashboard autonome et Solenda appellent le même moteur.
- Le moteur doit être multi-client / multi-tenant.

## Principe central

Le client fournit des **faits**. Le moteur produit les **déductions**.

Le client ne doit pas avoir à déclarer :
- « je peux stocker en Allemagne » ;
- « cette route est possible » ;
- « je suis couvert REP dans ce pays ».

Le moteur doit générer les routes candidates à partir des faits disponibles et des règles documentées.

## Données persistantes du profil client

Le profil contient principalement :
- entité vendeuse ;
- pays d'établissement ;
- pays où le client dispose d'une immatriculation TVA ;
- prep centers réellement disponibles ;
- paramètres Amazon/FBA explicitement connus lorsqu'ils sont disponibles automatiquement ou transmis par Solenda ;
- **immatriculations REP/EPR/IDU déjà détenues, lorsqu'elles existent et sont disponibles**.

Les immatriculations REP/EPR existantes sont **facultatives et non centrales dans l'UX V1**, mais le moteur doit pouvoir les exploiter pour déterminer qu'aucune **nouvelle** inscription n'est nécessaire lorsqu'une inscription propre, valide et applicable existe déjà.

La TVA est une donnée de profil utile, mais :
- elle ne prouve pas la conformité REP/EPR ;
- elle ne prouve pas à elle seule qu'un stockage FBA est possible ;
- elle ne doit pas servir de raccourci réglementaire.

## Données transactionnelles

À chaque analyse, fournir ou enrichir :
- EAN / GTIN / UPC / ASIN ;
- données produit disponibles ;
- pays de sourcing / pays réel du stock fournisseur ;
- marketplace cible ;
- fournisseur lorsque connu.

Solenda pourra transmettre ces informations directement via l'API.

## Routes

Le moteur génère les routes candidates raisonnables à partir :
- du pays de stock fournisseur ;
- des prep centers réellement disponibles ;
- de la marketplace cible ;
- des faits Amazon/FBA explicitement connus ;
- des règles et contraintes documentées.

Ne pas coder une route unique.
Ne pas demander au client de construire lui-même le trajet.
Ne pas croiser arbitrairement tous les pays sans règle ou donnée justificative.

Le modèle doit toujours préserver séparément :
- stock fournisseur ;
- transit simple ;
- prep center ;
- stock FBA ;
- marketplace cible.

Un prep center doit être référencé par un identifiant du profil client. Un transit sans prep center doit pouvoir être représenté séparément.

## Granularité du résultat

La décision doit être calculée au minimum par :

> **route × marché cible × filière**

Un verdict global éventuel n'est qu'un résumé. Il ne doit jamais masquer les résultats détaillés.

Pour un produit multi-filières, une route ne peut être favorable que si toutes les filières applicables requises sont favorablement résolues.

Pour éviter tout faux OUI par omission, chaque route doit contenir une entrée pour chacune des filières V1 (PACKAGING, EEE/WEEE, BATTERY) avec `applicability = APPLICABLE | NOT_APPLICABLE | UNRESOLVED`. Une filière UNRESOLVED ou un périmètre global de filières non résolu force NON.

## UX autonome

L'analyse autonome doit rester simple.

Cible V1 :
1. identifiant produit ;
2. pays de sourcing / stock fournisseur ;
3. marketplace cible.

Une requête d'analyse porte sur une seule marketplace cible en V1. Les analyses multi-marchés sont obtenues par orchestration de plusieurs appels.

Les autres données doivent venir :
- du profil client ;
- des providers ;
- de Solenda ;
- ou être demandées uniquement lorsqu'elles sont réellement nécessaires.

## Verdict

L'utilisateur voit uniquement :
- **OUI**
- **NON**

Cette convention doit être utilisée uniformément dans les documents, l'API V1 et le dashboard.

Les états internes peuvent être :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

Un **NON** lié à un manque de preuves signifie seulement que le moteur ne peut pas démontrer que la route peut être poursuivie sans nouvel enregistrement.

Il ne signifie pas automatiquement :
- que la vente est juridiquement interdite ;
- que le vendeur doit certainement s'enregistrer ;
- qu'une non-conformité juridique est établie.

Aucun score de confiance visible.

## Amazon

- Amazon SP-API est facultative.
- Le moteur doit fonctionner sans compte Amazon Professional.
- Les données Amazon peuvent enrichir l'analyse.
- Amazon ne constitue jamais une preuve juridique REP/EPR.
- Le moteur ne doit pas exiger que l'utilisateur renseigne manuellement un « pays FBA possible » s'il peut le déduire ou si cette donnée n'est pas nécessaire à la règle évaluée.

## Responsabilité REP

Ne jamais appliquer :

```text
numéro REP trouvé = produit couvert
```

Le moteur doit distinguer :
1. produit identifié ;
2. marque / fabricant / titulaire identifiés ;
3. entité juridique locale résolue ;
4. enregistrement officiel vérifié ;
5. lien entre l'entité, la filière, le produit et le marché ;
6. applicabilité à la route analysée ;
7. inscription propre déjà valide du vendeur, lorsqu'elle existe.

## Gate avant tout OUI réel

Aucun OUI réel ne doit être produit pour une combinaison pays × filière × rôle tant que :
- la règle applicable n'est pas documentée ;
- la source ou preuve utilisée n'est pas auditée ;
- le comportement en cas de données manquantes n'est pas testé ;
- des cas négatifs et ambigus n'ont pas été couverts.

Le gate doit aussi empêcher le cas trivial d'un moteur qui retourne toujours NON : pour chaque combinaison pays × filière × rôle activée, le corpus contient au minimum 3 cas positifs indépendamment revus et 3 cas négatifs/ambigus, et le moteur doit reconnaître 100 % des cas positifs du corpus activé.

Cas minimaux à tester :
- pays FBA inconnu ;
- entité amont non résolue ;
- REP amont trouvée mais non liée au produit ;
- registre officiel indisponible ;
- produit multi-filières ;
- même GTIN vers plusieurs marchés ;
- plusieurs routes vers un même marché ;
- couverture amont valide ;
- inscription propre déjà valide ;
- preuve insuffisante.

## V1 ≠ avis juridique

L'application produit une **pré-validation automatisée, conservatrice et traçable**, pas un avis juridique définitif.
