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
- paramètres Amazon/FBA explicitement connus lorsqu'ils sont disponibles automatiquement ou transmis par Solenda.

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

## REP/EPR détenues par le client

Les enregistrements REP/EPR/IDU déjà détenus par le client **ne sont pas une donnée centrale de la V1**.

L'objectif principal est d'identifier les routes qui **évitent** d'imposer un nouvel enregistrement au client.

Le modèle peut rester extensible pour prendre en compte une immatriculation existante plus tard, mais elle ne doit pas structurer l'UX ni le moteur V1.

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

## UX autonome

L'analyse autonome doit rester simple.

Cible V1 :
1. identifiant produit ;
2. pays de sourcing / stock fournisseur ;
3. marketplace cible.

Les autres données doivent venir :
- du profil client ;
- des providers ;
- de Solenda ;
- ou être demandées uniquement lorsqu'elles sont réellement nécessaires.

## Verdict

L'utilisateur voit uniquement :
- **OUI**
- **NON**

Les états internes peuvent être :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

Un NON lié à un manque de preuve ne signifie pas « vente juridiquement interdite ». L'explication doit préciser ce qui manque.

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
6. applicabilité à la route analysée.

## V1 ≠ avis juridique

L'application produit une **pré-validation automatisée, conservatrice et traçable**, pas un avis juridique définitif.
