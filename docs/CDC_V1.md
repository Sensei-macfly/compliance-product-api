# Cahier des charges V1 — Compliance Product API

## 1. Objectif

Construire un moteur autonome de pré-validation REP/EPR permettant de répondre :

> **À partir des faits connus sur le produit, le sourcing, la marketplace cible et la configuration du client, quelles routes permettent de poursuivre la vente sans nouvel enregistrement REP/EPR/IDU de l'entité vendeuse ?**

Le projet est une brique indépendante destinée à être testée seule puis intégrée à Solenda via API. Il ne doit pas devenir une plateforme de sourcing.

## 2. Principe produit

Le client fournit des faits.
Le moteur fournit les déductions.

Ne pas demander au client de déclarer lui-même :
- les pays où il « peut » stocker ;
- les routes qu'il estime possibles ;
- sa conformité REP supposée.

Le moteur doit utiliser les données disponibles pour générer des routes candidates raisonnables et produire un verdict.

## 3. Deux modes d'utilisation

### Dashboard autonome

L'interface collecte les données minimales nécessaires.

Cible V1 :
1. identifiant produit : EAN / GTIN / UPC / ASIN ;
2. pays de sourcing / pays réel du stock fournisseur ;
3. marketplace cible.

Les autres informations viennent :
- du profil client ;
- des providers ;
- ou d'une saisie complémentaire seulement lorsqu'elle est réellement nécessaire.

### Solenda via API

Solenda appelle le même moteur et peut envoyer un payload enrichi contenant :
- identifiant produit ;
- marque / catégorie / ASIN ;
- pays de sourcing / stock fournisseur ;
- marketplace cible ;
- fournisseur ;
- configuration compte ;
- données Amazon/FBA déjà disponibles.

Le moteur ne doit jamais lire directement la base Solenda.

## 4. Profil client

Le profil persistant contient principalement :
- entité vendeuse ;
- pays d'établissement ;
- pays où le client dispose d'une immatriculation TVA ;
- prep centers réellement disponibles ;
- faits Amazon/FBA explicitement connus lorsqu'ils sont disponibles.

Les enregistrements REP/EPR déjà détenus par le client ne sont pas une donnée centrale de la V1.

Objectif principal : identifier les routes qui évitent une nouvelle immatriculation.

## 5. Rôle de la TVA

Les pays de TVA peuvent être utilisés comme information de routage ou de contrôle selon des règles documentées.

Ils ne prouvent jamais :
- la conformité REP/EPR ;
- la couverture par un tiers ;
- l'éligibilité FBA à eux seuls ;
- le pays réel de stockage Amazon.

## 6. Prep centers

Les prep centers sont des faits logistiques persistants.

Ils servent à construire des routes candidates.
Ils ne transfèrent pas une obligation REP/EPR et ne constituent pas une preuve de couverture.

## 7. Marketplace cible

La marketplace cible fait partie de l'analyse.

Marchés pris en charge V1 :
- FR
- DE
- IT
- ES
- GB

Le moteur peut analyser un ou plusieurs marchés cibles, mais une analyse Solenda peut porter sur une marketplace précise.

## 8. Amazon

Amazon SP-API est facultative.

Le MVP doit fonctionner sans :
- compte Amazon Professional ;
- credentials SP-API ;
- connexion Seller Central.

Amazon peut enrichir :
- ASIN ↔ GTIN ;
- titre ;
- marque ;
- manufacturer ;
- catégorie ;
- product type ;
- faits logistiques explicitement disponibles.

Amazon n'est jamais une preuve juridique REP/EPR.

## 9. Pipeline

```text
Input facts
→ Product Resolution
→ Brand / Entity Resolution
→ EPR Classification
→ Official Registry Matching
→ Route Generation
→ Country / Supply-chain Rules
→ Internal State
→ OUI / NON
→ Reasons + Evidence
```

## 10. Product Resolution

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

Un identifiant produit ne suffit pas à établir toutes les caractéristiques réglementaires.

## 11. Entity Resolution

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

Une société apparentée ne doit jamais être automatiquement considérée comme l'entité REP responsable.

## 12. Filières V1

Minimum :
- PACKAGING
- EEE/WEEE
- BATTERY

Modèle extensible.

## 13. Sources

Priorité aux sources officielles/gratuites :
- GS1 si accès exploitable ;
- GLEIF ;
- EUIPO ;
- API Recherche Entreprises ;
- INPI/RNE ;
- Companies House ;
- ADEME ;
- LUCID/ZSVR ;
- Stiftung EAR ;
- GOV.UK EPR/WEEE ;
- registres officiels Espagne ;
- registres officiels Italie.

Les providers payants ne doivent pas être indispensables en V1.

## 14. Route Generator

Le générateur utilise :
- pays de stock fournisseur ;
- marketplace cible ;
- prep centers disponibles ;
- faits Amazon/FBA explicitement connus ;
- pays de TVA comme contexte uniquement lorsque pertinent ;
- règles logistiques/réglementaires documentées.

Il doit :
- générer des routes candidates raisonnables ;
- ne pas exiger que l'utilisateur choisisse lui-même la route ;
- ne pas déduire un pays FBA depuis la seule TVA ;
- ne pas produire toutes les permutations de pays ;
- conserver les informations manquantes comme état interne UNVERIFIED.

## 15. Verdict

Sortie utilisateur :
- OUI
- NON

États internes :
- VERIFIED
- UNVERIFIED
- NON_COMPLIANT
- CONTRADICTORY

OUI nécessite une chaîne de preuves suffisante.

NON couvre tous les autres cas mais doit être expliqué.

Principe :
> **UNKNOWN ≠ COMPLIANT**

## 16. Couverture amont

Un numéro REP/IDU d'une marque, d'un fabricant ou d'une société liée n'est jamais suffisant seul.

Pour conclure favorablement, le moteur doit pouvoir établir, selon la règle applicable :
- identité de l'entité ;
- pays ;
- filière/catégorie ;
- validité de l'enregistrement ;
- lien pertinent avec le produit / la catégorie ;
- applicabilité à la route et au marché.

## 17. API

### POST /api/v1/analyses

Payload minimal :

```json
{
  "identifier": "8710103995426",
  "supplier_stock_country": "IT",
  "target_marketplace": "DE"
}
```

Payload enrichi possible :

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
    "vat_countries": ["FR", "DE"],
    "prep_centers": [
      {"country": "FR"}
    ]
  }
}
```

Les champs enrichis sont facultatifs.

## 18. Réponse API

```json
{
  "analysis_id": "uuid",
  "verdict": "NO",
  "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE",
  "missing_context": [],
  "routes": [],
  "evidence": []
}
```

Le même moteur doit répondre au dashboard autonome et à Solenda.

## 19. Reason codes minimaux

- ALL_REQUIRED_STREAMS_VERIFIED
- NO_LOCAL_ENTITY_FOUND
- EPR_REGISTRATION_NOT_FOUND
- MULTIPLE_STREAMS_INCOMPLETE
- CROSS_BORDER_COVERAGE_NOT_ESTABLISHED
- PRODUCT_CLASSIFICATION_UNCERTAIN
- ENTITY_RESOLUTION_FAILED
- OFFICIAL_REGISTRY_UNAVAILABLE
- INSUFFICIENT_OFFICIAL_EVIDENCE
- PRODUCT_NOT_RESOLVED
- UPSTREAM_COVERAGE_VERIFIED
- UPSTREAM_COVERAGE_UNPROVEN
- ROUTE_NOT_SUPPORTED
- MARKET_RULE_NOT_VALIDATED
- REGISTRATION_SCOPE_MISMATCH
- REQUIRED_ROUTE_FACT_MISSING

## 20. Dashboard

Afficher :
- produit ;
- sourcing / stock fournisseur ;
- marketplace cible ;
- routes candidates ;
- OUI/NON ;
- motifs ;
- filières ;
- entités ;
- numéros/enregistrements ;
- preuves ;
- limites.

Ne pas demander de configuration logistique détaillée à chaque analyse si elle existe déjà dans le profil.

## 21. Données

PostgreSQL + Prisma.

Tables principales :
- users
- client_profiles
- client_vat_countries
- user_prep_centers
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
- candidate_routes
- route_results
- evidence

## 22. Règles minimales

- filière obligatoire non vérifiée => NON ;
- entité nécessaire non résolue => NON ;
- registre indisponible sans cache récent => NON ;
- numéro REP de marque seul => insuffisant ;
- numéro d'un autre pays => insuffisant ;
- achat dans un pays ≠ couverture automatique dans un autre ;
- toutes les filières requises doivent être traitées ;
- une donnée de route manquante ne doit bloquer que si la règle évaluée en dépend.

## 23. Hors périmètre V1

Ne pas implémenter :
- plateforme de sourcing ;
- comparaison fournisseurs ;
- ROI ;
- marge FBA ;
- fiscalité complète ;
- moteur logistique/TMS exhaustif ;
- inscription REP automatisée ;
- APIs commerciales obligatoires ;
- avis juridique définitif.

## 24. Critères d'acceptation

La V1 est fonctionnelle si :
1. elle accepte un identifiant produit ;
2. elle accepte un pays de stock fournisseur ;
3. elle accepte une marketplace cible ;
4. elle exploite le profil client sans lui demander de deviner ses routes ;
5. elle génère des routes candidates ;
6. elle résout produit et entités ;
7. elle classe les filières ;
8. elle interroge/synchronise les sources officielles ;
9. elle retourne OUI/NON avec motifs et preuves ;
10. les champs enrichis Solenda restent optionnels ;
11. le dashboard et Solenda utilisent le même moteur ;
12. aucune connexion Amazon n'est obligatoire ;
13. aucune API commerciale payante n'est indispensable.
