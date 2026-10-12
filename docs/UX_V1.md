# UX V1 — principes d'interface

## Direction visuelle

Direction retenue : **Style C — SaaS premium + clarté opérationnelle**.

Principes :
- interface très propre et professionnelle ;
- hiérarchie visuelle forte ;
- informations réglementaires importantes visibles sans clic inutile ;
- peu de couleurs, utilisées avec une sémantique stable ;
- pas de dashboard surchargé ;
- pas de graphiques décoratifs inutiles ;
- pas de score de confiance visible ;
- pas de formulation « conforme » trop absolue quand le moteur vérifie en réalité une couverture amont.

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
- libellé principal : « Couverture amont vérifiée » ;
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

Libellé principal recommandé :
> **Couverture complète non démontrée**

Exemple :
- PACKAGING : numéro officiel vérifié + couverture amont vérifiée ;
- EEE/WEEE : numéro officiel trouvé mais couverture du produit non démontrée ;
- BATTERY : non applicable.

Le blocage EEE/WEEE est mis en évidence, mais le numéro PACKAGING reste visible.

Un NON causé par un manque de preuve utilise une présentation d'alerte/ambre plutôt qu'un écran intégralement rouge.

Le rouge est réservé à une contradiction, un enregistrement explicitement invalide ou une non-conformité réellement établie.

## Cartes filières

Pour chaque filière applicable, distinguer explicitement :

```text
Numéro officiel
✓ Vérifié / À vérifier

Couverture de ce produit
✓ Vérifiée / Non démontrée
```

Les numéros d'enregistrement doivent être visibles directement dans la carte lorsqu'ils sont disponibles.

## Ce qui est vérifié / Point à vérifier

Le résultat doit comporter deux zones faciles à lire :

### Ce qui est vérifié

Exemples :
- produit identifié ;
- marque identifiée ;
- filières applicables déterminées ;
- entité titulaire identifiée ;
- numéro officiel vérifié ;
- registre officiel consulté.

### Point à vérifier

Expliquer en français naturel le fait précis qui empêche actuellement un OUI.

Exemple :
> Un numéro WEEE officiel a été trouvé et vérifié, mais nous n'avons pas encore pu démontrer que cet enregistrement couvre précisément ce produit ou cette marque.

## Preuves et détails techniques

Les preuves et sources sont disponibles dans une section dépliable.

Les détails techniques sont repliés par défaut et peuvent contenir :
- `internal_state` ;
- `reason_codes` ;
- `stream_scope_state` ;
- informations de matching ;
- métadonnées de sources.

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

Sémantique recommandée :
- OUI / vérifié : vert ;
- NON dû à preuve insuffisante / à vérifier : ambre ;
- contradiction ou élément explicitement invalide : rouge ;
- information / action primaire : bleu ;
- non applicable : neutre.

Un NON ne doit pas être présenté automatiquement comme « vente interdite ».

## Responsive V1 — desktop, iPhone et Android

La compatibilité mobile est une **exigence V1**, pas une amélioration ultérieure.

La V1 doit être pleinement utilisable depuis :
- desktop ;
- tablette ;
- Safari sur iPhone ;
- Chrome sur Android.

Il s'agit d'une application web responsive ; une application iOS/Android native n'est pas requise pour la V1.

### Principes mobile

- aucun scroll horizontal sur les écrans métier ;
- aucune interaction essentielle dépend du hover ;
- zones tactiles d'au moins 44 × 44 px ;
- formulaires utilisables au pouce ;
- bouton principal pleine largeur sur petit écran lorsque pertinent ;
- support des safe areas iPhone ;
- gestion correcte du clavier virtuel ;
- textes et numéros réglementaires lisibles sans zoom ;
- accordéons, boutons de copie et liens utilisables au toucher ;
- navigation simplifiée, sans sidebar desktop écrasée sur mobile.

### Adaptation de l'écran d'analyse

Desktop :
```text
[ Identifiant ] [ Pays stock ] [ Marketplace ] [ Analyser ]
```

Mobile :
```text
Identifiant
[........................]

Pays du stock fournisseur
[........................]

Marketplace cible
[........................]

[      Analyser le produit      ]
```

### Adaptation de l'écran résultat

Desktop :
```text
[ Emballages ] [ DEEE ] [ Batteries ]
[ Ce qui est vérifié ] [ Point à vérifier ]
```

Mobile :
```text
[ Verdict ]

[ Point à vérifier ]   ← si verdict NON

[ Emballages ]
[ DEEE ]
[ Batteries ]

[ Ce qui est vérifié ]

[ Preuves et sources ]
[ Détails techniques ]
```

Sur un NON, le point bloquant principal doit apparaître immédiatement après le verdict, avant les cartes filières, afin de rester visible sans long scroll. Les éléments déjà vérifiés restent ensuite pleinement visibles.

### Navigation mobile

Sur petit écran :
- header compact ;
- menu mobile ou navigation réduite ;
- pas de sidebar permanente ;
- actions secondaires regroupées dans un menu ;
- action principale clairement accessible.

### Breakpoints indicatifs

- mobile compact : 320–479 px ;
- mobile large : 480–767 px ;
- tablette : 768–1023 px ;
- desktop : ≥ 1024 px.

Le design doit rester fluide entre les breakpoints ; il ne doit pas dépendre uniquement de largeurs fixes.

### Compatibilité navigateur à tester

Minimum V1 :
- Safari iOS récent ;
- Chrome Android récent ;
- Chrome desktop récent ;
- Safari desktop récent ;
- Edge récent.

### Tests responsive

Prévoir des tests Playwright au minimum sur des profils équivalents à :
- iPhone compact ;
- iPhone récent grand format ;
- Pixel / Android standard ;
- tablette ;
- desktop 1280 px ou plus.

Critères :
- pas de contenu coupé ;
- pas de scroll horizontal ;
- verdict visible et lisible ;
- cartes filières correctement empilées ;
- numéros copiables ;
- accordéons utilisables ;
- formulaire entièrement utilisable avec clavier tactile.

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
