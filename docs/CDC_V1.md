# Cahier des charges V1 — Compliance Product API

## 1. Objectif

Construire un moteur autonome de pré-validation REP/EPR qui répond, pour **un produit et un marché cible** :

> **Peut-on démontrer qu'une couverture REP/EPR amont suffisante existe, de sorte que l'entité vendeuse n'ait pas à porter elle-même une nouvelle obligation REP/EPR pour les filières applicables ?**

L'objectif métier est d'identifier les cas où le vendeur peut s'appuyer légalement sur une couverture amont déjà existante. La V1 ne promet pas une absence absolue de coûts ou d'obligations annexes : elle vérifie la possibilité d'éviter une obligation REP propre au vendeur pour le scénario analysé.

## 2. Principe central

Le client fournit des **faits**. Le moteur produit les **déductions**.

Le moteur ne doit pas demander au client de deviner :
- un pays FBA ;
- une route logistique ;
- un prep center ;
- une conformité REP supposée.

## 3. Entrées V1

### Analyse

Une requête analyse une seule marketplace cible.

Entrées minimales :
1. `identifier` : EAN / GTIN / UPC / ASIN ;
2. `supplier_stock_country` : pays où le produit est **physiquement stocké chez le fournisseur au moment du sourcing** ;
3. `target_marketplace` : FR / DE / IT / ES / GB.

`supplier_stock_country` ne signifie pas :
- pays de facturation ;
- pays d'immatriculation du fournisseur ;
- pays supposé d'expédition si le stock réel est ailleurs.

### Profil client

Faits persistants minimaux :
- entité vendeuse ;
- pays d'établissement ;
- pays d'immatriculation TVA si disponibles comme contexte.

Les immatriculations REP/EPR propres du vendeur ne constituent pas une voie favorable recherchée par la V1.

## 4. Hors périmètre logistique V1

La V1 ne modélise pas :
- prep centers ;
- transit ;
- route physique ;
- pays de stockage FBA ;
- redistribution du stock par Amazon ;
- optimisation transport / coûts / délais.

Raison : au moment du sourcing et de l'achat, le pays de stockage FBA futur n'est généralement pas connu de manière fiable. Le moteur ne doit jamais l'inventer.

## 5. Hypothèse produit V1

La V1 suppose :
- produit revendu dans son état commercial d'origine ;
- pas de bundle créé par le vendeur ;
- pas de reconditionnement ;
- pas de modification d'emballage susceptible de créer une nouvelle obligation.

Ces cas sont réservés à une V2.

## 6. Amazon

Amazon est le canal de vente, pas une source de preuve juridique REP/EPR.

Amazon SP-API est facultative et peut uniquement enrichir :
- ASIN ↔ GTIN ;
- titre ;
- marque ;
- manufacturer ;
- catégorie ;
- product type.

La V1 ne dépend pas :
- d'un compte Amazon Professional ;
- des pays de stockage FBA ;
- d'un shipment plan ;
- d'une prédiction de redistribution Amazon.

## 7. Pipeline V1

```text
Input facts
→ Product Resolution
→ Brand / Entity Resolution
→ EPR Classification
→ Target-market Rules
→ Official Registry Matching
→ Upstream Coverage Assessment
→ Stream Results
→ OUI / NON
→ French Explanation + Evidence
```

## 8. Product Resolution

Objet minimal :

```json
{
  "input_identifier": "8710103995426",
  "identifier_type": "EAN13",
  "gtin": "8710103995426",
  "asin": null,
  "title": null,
  "brand": null,
  "manufacturer": null,
  "category": null,
  "product_type": null,
  "image_url": null
}
```

Un identifiant produit ne suffit jamais, seul, à établir la responsabilité REP.

## 9. Entity Resolution

Distinguer :
- marque ;
- propriétaire de marque ;
- titulaire GTIN ;
- fabricant ;
- groupe parent ;
- filiale locale ;
- distributeur ;
- importateur ;
- producteur REP enregistré.

Une société liée au groupe ne doit pas être considérée automatiquement comme l'entité couvrant le produit.

## 10. Filières V1

Univers minimal :
- PACKAGING
- EEE/WEEE
- BATTERY

Pour chaque analyse, `streams[]` doit représenter toutes les filières V1.

`applicability` :
- `APPLICABLE`
- `NOT_APPLICABLE`
- `UNRESOLVED`

Règles :
- `APPLICABLE` : nécessite une couverture amont vérifiée pour obtenir OUI ;
- `NOT_APPLICABLE` : ne bloque pas le verdict global ;
- `UNRESOLVED` : force NON ;
- aucune filière ne peut être omise pour produire un OUI.

`stream_scope_state` :
- `COMPLETE`
- `UNRESOLVED`

`UNRESOLVED` force le verdict global à NON.

## 11. Couverture amont

Un numéro trouvé n'est jamais suffisant à lui seul.

Pour produire OUI sur une filière applicable, le moteur doit pouvoir démontrer, selon la règle du marché cible :
- l'identité de l'entité amont pertinente ;
- son rôle pertinent ;
- le bon pays ;
- la bonne filière / catégorie ;
- la validité de l'enregistrement officiel ;
- le lien avec la marque / le produit / la catégorie lorsque la règle l'exige ;
- l'applicabilité de cette couverture à la vente analysée.

La conclusion favorable recherchée est :

> **UPSTREAM_COVERAGE_VERIFIED**

Une immatriculation propre du vendeur ne transforme pas à elle seule le résultat en OUI : ce n'est pas le scénario d'optimisation recherché par la V1.

## 12. Verdict

Sortie utilisateur :
- **OUI**
- **NON**

États internes possibles :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

### OUI

Pour une filière :
- `NOT_APPLICABLE`, ou
- `APPLICABLE` + couverture amont vérifiée.

Pour l'analyse globale :
- `stream_scope_state = COMPLETE` ;
- toutes les filières V1 ont `verdict = OUI`.

### NON

NON signifie qu'une couverture suffisante n'est pas démontrée pour l'ensemble du scénario.

NON ne signifie pas automatiquement :
- vente interdite ;
- inscription certaine obligatoire ;
- non-conformité juridiquement établie.

Principe :
> **UNKNOWN ≠ COMPLIANT**

## 13. Contrat API V1 de référence

Cette section est la **source normative unique** du contrat V1.

### Requête minimale

```json
{
  "identifier": "8710103995426",
  "supplier_stock_country": "IT",
  "target_marketplace": "DE"
}
```

### Requête enrichie

```json
{
  "identifier": "8710103995426",
  "supplier_stock_country": "IT",
  "target_marketplace": "DE",
  "product": {
    "asin": "B0XXXX",
    "brand": "Example",
    "category": "Electric Toothbrush"
  },
  "supplier": {
    "legal_name": "Example Supplier",
    "country": "IT"
  },
  "account_context": {
    "seller_country": "FR",
    "vat_countries": ["FR", "DE"]
  }
}
```

### Réponse

```json
{
  "analysis_id": "uuid",
  "supplier_stock_country": "IT",
  "target_marketplace": "DE",
  "overall_verdict": "NON",
  "stream_scope_state": "COMPLETE",
  "streams": [
    {
      "stream": "PACKAGING",
      "applicability": "APPLICABLE",
      "verdict": "OUI",
      "coverage_status": "UPSTREAM_VERIFIED",
      "internal_state": "VERIFIED",
      "registrations": [
        {
          "registration_type": "PACKAGING",
          "registration_number": "DE123456789",
          "holder": "Example GmbH",
          "country": "DE",
          "registry": "OFFICIAL_REGISTRY",
          "registration_status": "VERIFIED",
          "product_coverage_status": "VERIFIED"
        }
      ],
      "reason_codes": ["UPSTREAM_COVERAGE_VERIFIED"],
      "user_message_fr": "Couverture amont vérifiée pour les emballages.",
      "evidence": []
    },
    {
      "stream": "EEE/WEEE",
      "applicability": "APPLICABLE",
      "verdict": "NON",
      "coverage_status": "UNPROVEN",
      "internal_state": "UNVERIFIED",
      "registrations": [
        {
          "registration_type": "WEEE",
          "registration_number": "DE87654321",
          "holder": "Example GmbH",
          "country": "DE",
          "registry": "OFFICIAL_REGISTRY",
          "registration_status": "VERIFIED",
          "product_coverage_status": "UNPROVEN"
        }
      ],
      "reason_codes": ["REGISTRATION_FOUND_COVERAGE_UNPROVEN"],
      "user_message_fr": "Un numéro officiel a été trouvé, mais sa couverture de ce produit n'est pas encore démontrée.",
      "evidence": []
    },
    {
      "stream": "BATTERY",
      "applicability": "NOT_APPLICABLE",
      "verdict": "OUI",
      "coverage_status": "NOT_REQUIRED",
      "internal_state": "VERIFIED",
      "registrations": [],
      "reason_codes": ["STREAM_NOT_APPLICABLE"],
      "user_message_fr": "Cette filière n'est pas applicable au produit analysé.",
      "evidence": []
    }
  ]
}
```

## 14. Invariants du contrat

- toutes les filières V1 figurent dans `streams[]` ;
- une filière applicable n'est OUI que si sa couverture amont est vérifiée ;
- un numéro peut être retourné même lorsque le verdict de la filière est NON ;
- `registration_status = VERIFIED` ne signifie pas `product_coverage_status = VERIFIED` ;
- une filière UNRESOLVED force NON ;
- un scope de filières UNRESOLVED force NON ;
- l'immatriculation propre du vendeur n'est pas utilisée comme raccourci pour produire OUI.

## 15. Reason codes minimaux

Codes internes en anglais :
- ALL_APPLICABLE_STREAMS_UPSTREAM_COVERED
- STREAM_NOT_APPLICABLE
- STREAM_APPLICABILITY_UNRESOLVED
- EPR_STREAM_SCOPE_UNRESOLVED
- UPSTREAM_COVERAGE_VERIFIED
- UPSTREAM_COVERAGE_UNPROVEN
- UPSTREAM_ENTITY_NOT_RESOLVED
- REGISTRATION_NOT_FOUND
- REGISTRATION_FOUND_COVERAGE_UNPROVEN
- REGISTRATION_SCOPE_MISMATCH
- PRODUCT_CLASSIFICATION_UNCERTAIN
- PRODUCT_NOT_RESOLVED
- OFFICIAL_REGISTRY_UNAVAILABLE
- TARGET_MARKET_RULE_NOT_VALIDATED
- SUPPLIER_STOCK_COUNTRY_UNCERTAIN
- INSUFFICIENT_OFFICIAL_EVIDENCE

L'interface utilisateur affiche des explications en français et ne montre pas les codes bruts par défaut.

## 16. Dashboard

Le résultat doit montrer immédiatement :
1. OUI / NON ;
2. numéros REP/EPR/DEEE trouvés ;
3. ce qui est vérifié ;
4. ce qui bloque ou reste à vérifier ;
5. les preuves et sources.

Même sur un résultat NON, les éléments correctement vérifiés doivent rester visibles.

Le dashboard V1 doit être entièrement responsive et conserver la totalité des fonctions métier sur :
- desktop ;
- tablette ;
- iPhone / Safari ;
- Android / Chrome.

Une application native iOS/Android n'est pas requise pour la V1.

Voir `docs/UX_V1.md`.

## 17. Données principales

PostgreSQL + Prisma.

Tables principales :
- users
- client_profiles
- client_vat_countries
- products
- product_identifiers
- brands
- entities
- entity_aliases
- entity_relationships
- epr_streams
- epr_registrations
- product_epr_classifications
- analyses
- analysis_stream_results
- registry_matches
- evidence

Pas de tables V1 pour :
- prep centers ;
- routes logistiques ;
- segments transit ;
- stockage FBA.

## 18. Règles minimales

- filière applicable non vérifiée => NON ;
- applicabilité non résolue => NON ;
- périmètre des filières non résolu => NON ;
- entité amont nécessaire non résolue => NON ;
- registre indisponible sans cache suffisamment récent => NON ;
- numéro officiel trouvé seul => insuffisant ;
- numéro d'un autre pays => insuffisant sauf règle explicitement documentée ;
- achat / stock dans un pays ≠ couverture automatique dans le marché cible ;
- une inscription propre du vendeur ≠ couverture amont favorable.

## 19. Hors périmètre V1

- plateforme de sourcing ;
- comparaison fournisseurs ;
- ROI / marge FBA ;
- moteur logistique ;
- prep centers ;
- transit ;
- pays de stockage FBA ;
- bundles ;
- reconditionnement ;
- modification d'emballage ;
- inscription REP automatisée ;
- fiscalité complète ;
- avis juridique définitif.

## 20. Gate avant activation de vrais OUI

Pour chaque combinaison pays × filière × rôle activée :
- corpus figé avant évaluation ;
- 0 faux OUI sur le corpus ;
- 100 % des OUI traçables jusqu'aux sources et à la version de règle ;
- au moins 3 cas positifs indépendamment revus ;
- au moins 3 cas négatifs ou ambigus ;
- 100 % des cas positifs de référence reconnus comme OUI ;
- source indisponible / preuve amont incomplète / applicabilité non résolue => NON/UNVERIFIED.

Ce minimum est acceptable pour un pilote limité, pas pour une validation à grande échelle. Le corpus devra être élargi avant extension du périmètre.

## 21. Critères d'acceptation V1

La V1 est fonctionnelle si :
1. elle accepte identifiant + pays physique du stock fournisseur + marketplace cible ;
2. elle fonctionne sans pays FBA ;
3. elle fonctionne sans prep center ;
4. elle résout produit et entités ;
5. elle classe PACKAGING / EEE-WEEE / BATTERY ;
6. elle recherche les enregistrements officiels du marché cible ;
7. elle distingue numéro trouvé et couverture produit démontrée ;
8. elle retourne tous les numéros pertinents trouvés, y compris lorsque le verdict global est NON ;
9. elle produit OUI uniquement avec couverture amont démontrée pour toutes les filières applicables ;
10. elle produit des explications utilisateur en français ;
11. les codes techniques restent en anglais ;
12. dashboard et Solenda utilisent le même moteur et le même contrat ;
13. Amazon SP-API n'est pas obligatoire ;
14. le dashboard reste pleinement fonctionnel sans scroll horizontal sur iPhone et Android, avec les mêmes informations et actions essentielles que sur desktop.
