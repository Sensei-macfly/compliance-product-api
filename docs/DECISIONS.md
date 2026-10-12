# Décisions produit — V1

Ce document fixe les décisions validées.

## Positionnement

- MVP autonome de compliance REP/EPR.
- Pas de plateforme de sourcing.
- Pas de moteur logistique.
- API-first.
- Dashboard autonome et Solenda utilisent le même moteur.
- Multi-client / multi-tenant.

## Objectif favorable

Le résultat favorable recherché est :

> **couverture amont vérifiée dans le marché cible, permettant de ne pas faire reposer l'obligation REP propre sur le vendeur pour la filière analysée.**

Une immatriculation propre du vendeur n'est pas considérée comme le scénario favorable recherché par la V1.

## Entrées minimales

1. identifiant produit ;
2. pays où le produit est physiquement stocké chez le fournisseur ;
3. marketplace cible.

Le pays de stock fournisseur n'est pas :
- le pays de facture ;
- le pays du siège du fournisseur ;
- une supposition.

## Suppression du concept de route V1

La V1 ne construit plus de route logistique.

Supprimés :
- prep center ;
- transit ;
- route segments ;
- pays FBA supposé ;
- scénario de stockage FBA ;
- prédiction Amazon.

Le moteur ne doit pas essayer de deviner ce qu'Amazon fera du stock.

## Hypothèse produit

V1 :
- produit inchangé ;
- emballage commercial d'origine ;
- pas de bundle ;
- pas de reconditionnement ;
- pas de modification d'emballage.

Bundle / repackaging / prep center à impact réglementaire : V2.

## Granularité du résultat

> **marché cible × filière**

Filières V1 :
- PACKAGING
- EEE/WEEE
- BATTERY

Chaque filière est toujours représentée.

`applicability = APPLICABLE | NOT_APPLICABLE | UNRESOLVED`.

Une applicabilité non résolue force NON.

## Couverture

Ne jamais appliquer :

```text
numéro trouvé = produit couvert
```

Le moteur doit distinguer :
1. numéro trouvé ;
2. titulaire du numéro ;
3. pays ;
4. filière / catégorie ;
5. validité ;
6. relation avec le produit / marque ;
7. applicabilité au marché cible ;
8. couverture amont démontrée ou non.

## Verdict

Utilisateur :
- OUI
- NON

OUI sur une filière applicable uniquement avec `UPSTREAM_COVERAGE_VERIFIED`.

Overall OUI uniquement si toutes les filières V1 sont OUI et le scope est complet.

Un NON lié à l'incertitude ne signifie pas automatiquement vente interdite ou obligation certaine de s'enregistrer.

## Interface

Le verdict doit être immédiatement visible.

Même en cas de NON :
- afficher tous les numéros officiels pertinents trouvés ;
- afficher les éléments déjà validés ;
- mettre en évidence le point bloquant ou à vérifier.

Les messages utilisateur sont en français.

Les reason codes et états techniques restent en anglais.

## Amazon

- Amazon SP-API facultative.
- Enrichissement produit uniquement en V1.
- Aucun pays FBA requis.
- Aucun shipment plan requis.
- Aucune prédiction de stockage Amazon.

## Gate avant OUI réel

Par combinaison pays × filière × rôle activée :
- règle documentée ;
- source auditée ;
- corpus figé ;
- 0 faux OUI ;
- 100 % des OUI traçables ;
- minimum 3 cas positifs indépendamment revus ;
- minimum 3 cas négatifs/ambigus ;
- 100 % des cas positifs de référence reconnus.

Ce seuil est un minimum de pilote, à augmenter avant extension.

## V1 ≠ avis juridique

Le moteur produit une pré-validation conservatrice, explicable et traçable.
