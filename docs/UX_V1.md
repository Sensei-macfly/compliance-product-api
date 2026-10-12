# UX V1 — principes d'interface

## Objectif des 5 premières secondes

L'utilisateur doit comprendre immédiatement :
1. le verdict global OUI / NON ;
2. les numéros REP/EPR/DEEE trouvés ;
3. ce qui est déjà vérifié ;
4. ce qui empêche un OUI ;
5. ce qu'il faut vérifier ensuite.

## Hiérarchie recommandée

```text
Produit + marché analysé
↓
OUI / NON très visible
↓
Numéros et couvertures trouvés
↓
Éléments validés / points bloquants
↓
Détail par filière
↓
Preuves / sources
↓
Détails techniques
```

## OUI

Afficher immédiatement :
- verdict OUI ;
- filières applicables ;
- numéros officiels pertinents ;
- titulaire ;
- registre ;
- pays ;
- statut de l'enregistrement ;
- statut de couverture du produit.

Ne jamais afficher seulement « numéro trouvé = OK ».

## NON

Un NON ne masque pas les résultats partiellement positifs.

Exemple :
- PACKAGING : numéro officiel vérifié + couverture amont vérifiée ;
- EEE/WEEE : numéro officiel trouvé mais couverture du produit non démontrée ;
- BATTERY : non applicable.

Le blocage EEE/WEEE est mis en évidence, mais le numéro PACKAGING reste visible.

## Vocabulaire utilisateur

Messages en français.

Exemples :

| Code interne | Message utilisateur |
|---|---|
| OFFICIAL_REGISTRY_UNAVAILABLE | Registre officiel temporairement indisponible. La vérification n'a pas pu être terminée. |
| PRODUCT_CLASSIFICATION_UNCERTAIN | Classification du produit à confirmer. Les filières applicables ne sont pas encore déterminées avec certitude. |
| UPSTREAM_COVERAGE_UNPROVEN | Couverture amont non démontrée pour cette filière. |
| REGISTRATION_FOUND_COVERAGE_UNPROVEN | Un numéro officiel a été trouvé, mais sa couverture de ce produit n'est pas encore démontrée. |
| REGISTRATION_SCOPE_MISMATCH | L'enregistrement trouvé ne correspond pas au périmètre nécessaire pour cette analyse. |
| REGISTRATION_NOT_FOUND | Aucun enregistrement correspondant n'a été trouvé dans le registre officiel consulté. |

Les codes anglais restent disponibles dans les détails techniques / API, pas dans l'interface normale.

## Couleurs / sévérité

Le design visuel final sera défini séparément, mais la sémantique doit rester :
- OUI : favorable démontré ;
- NON/UNVERIFIED : couverture insuffisamment démontrée ;
- contradiction explicite : signal plus fort distinct.

Un NON ne doit pas être présenté automatiquement comme « vente interdite ».

## Pas de route logistique dans l'écran V1

Ne pas afficher :
- transit ;
- prep center ;
- pays FBA supposé ;
- route physique inventée.

Afficher seulement les faits V1 :
- pays du stock fournisseur ;
- marché cible ;
- vendeur ;
- produit.
