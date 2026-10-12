# Cahier des charges V1 — Compliance Product API

## 1. Objectif

Construire un MVP autonome qui vérifie, pour un parcours REP/EPR précisément défini, les obligations et preuves identifiables à partir de sources auditables. La V1 ne doit pas prétendre autoriser globalement la vente d'un produit.

> **À partir du produit, des pays source/stock et du profil client, quelles routes vers les marketplaces cibles peuvent être poursuivies sans nouvel enregistrement REP/EPR/IDU de l'entité vendeuse évaluée ?**

Le moteur doit générer les routes candidates à partir des données et contraintes du client ; aucune route unique ne doit être codée en dur. Le pilote valide cette génération et la vérification REP/EPR, sans devenir une plateforme de sourcing.

## 2. Entrées

Le modèle de départ est un vendeur Amazon qui achète auprès de fournisseurs/grossistes de plusieurs pays européens, puis peut faire transiter le stock par des prep centers avant Amazon FBA et la vente sur plusieurs marketplaces. Le moteur doit être multi-client : les mêmes règles s'appliquent à l'entité légale de chaque client, sans supposer qu'il s'agit toujours de la société du créateur du projet.

Deux champs avaient été proposés pour l'expérience initiale, mais ils ne suffisent pas à établir à qui incombe l'obligation REP/EPR :
1. EAN / GTIN / UPC / ASIN, comme aide à l'identification — pas comme preuve de catégorie ou de couverture ;
2. pays où le produit est physiquement stocké chez le fournisseur, comme contexte logistique — pas comme preuve du rôle REP de chaque acteur.

Les faits d'entrée doivent permettre de générer les routes candidates :
- le client/tenant, son entité vendeuse et le pays d'établissement ;
- les pays où le client dispose d'une immatriculation TVA, comme contrainte de profil à interpréter — jamais comme preuve REP/EPR ;
- les enregistrements REP/EPR déjà détenus par le client et leurs pays/filières/catégories ;
- le fournisseur/grossiste, son pays d'établissement et les pays de stock/source disponibles ;
- les prep centers réellement disponibles pour ce client et leurs pays ;
- le ou les programmes FBA configurés et les pays d'inventaire permis ou connus ;
- les marketplaces cibles demandées ;
- le produit et ses attributs de classification ;
- le fabricant/producteur ou l'autre entité amont qui est prétendue responsable REP/EPR, son identifiant juridique et ses enregistrements ;
- les pièces reliant cette entité au client, au produit/SKU, à la filière/catégorie et au marché concerné.

Le générateur croise ces faits pour construire les routes plausibles, sans inventer d'étapes absentes des données ni demander à l'utilisateur de choisir une route codée d'avance. Les contraintes d'éligibilité opérationnelle Amazon/FBA doivent venir du programme ou des données enrichies du client ; une marketplace cible, un pays de TVA ou un prep center disponible ne prouvent pas à eux seuls le pays réel de stockage FBA.

L'identifiant produit (EAN/GTIN/ASIN) et les attributs nécessaires à la classification restent requis, mais ne remplacent aucun de ces faits de parcours. Une absence d'information pertinente produit la décision utilisateur **NON** ; en interne, elle reste NON VÉRIFIÉE, sans déduction à partir du pays du fournisseur.

Le pays d'établissement du fournisseur, le pays de stock initial, les pays de transit/stockage et le marché cible sont des axes distincts. Ne pas supposer qu'un prep center, une immatriculation TVA ou une immatriculation amont transfère automatiquement la couverture REP/EPR. Ne pas demander un numéro de TVA par principe : le profil peut fournir les seuls pays où le client est immatriculé ; demander un identifiant légal seulement si nécessaire à la résolution d'une entité dans la source utilisée. Pour les entités étrangères, la source ADEME peut utiliser un identifiant fiscal étranger.

Pays possibles :
- France
- Allemagne
- Italie
- Espagne
- Royaume-Uni
- Autre UE
- Hors UE
- Inconnu

## 3. Marchés analysés

Marchés pris en charge par la V1 :
- FR
- DE
- IT
- ES
- GB

FR, DE, IT, ES et GB sont les marchés pris en charge par la V1 ; l'application appelante transmet les marchés cibles demandés parmi ceux-ci. Les pays fournisseurs/stock peuvent inclure d'autres pays de l'UE. Produire une analyse pour chaque combinaison de parcours d'approvisionnement/stockage pris en charge et marché cible demandé. Les pays ou filières sans règles et preuves auditées retournent **NON** à l'utilisateur, jamais un OUI implicite. Ajouter d'autres marchés de vente exige une décision de périmètre.

La source ADEME France / EEE est un candidat pour vérifier une pièce d'enregistrement française. Elle ne valide pas à elle seule un parcours Amazon FBA transfrontalier, la responsabilité de la société vendeuse ou une couverture dans les autres pays.

## 4. Verdict

Les états ci-dessous sont **internes** ; ils ne constituent pas des choix supplémentaires dans l'interface utilisateur :
- **VÉRIFIÉ** : les preuves requises pour le parcours REP/EPR défini sont suffisantes et cohérentes ;
- **NON CONFORME** : une obligation applicable est établie et des preuves suffisantes montrent qu'elle n'est pas remplie ;
- **NON VÉRIFIÉ** : informations, correspondance d'entité, couverture de source ou preuves insuffisantes ;
- **CONTRADICTOIRE** : des sources pertinentes se contredisent.

L'interface n'affiche que **OUI / NON** à la question « peut-on poursuivre cette route sans nouvel enregistrement de l'entité vendeuse évaluée ? ». OUI exige des preuves suffisantes et cohérentes. NON couvre tous les autres cas, y compris les preuves manquantes ou contradictoires. L'explication peut décrire le blocage, mais ne présente jamais NON VÉRIFIÉ comme une troisième décision commerciale.

En interne, chaque résultat précise la situation d'inscription, sans déduire une dispense de la seule présence d'un IDU amont :
- **SELLER_REGISTRATION_MISSING** : une règle applicable établit que l'entité vendeuse évaluée doit être enregistrée, mais la preuve d'un enregistrement valable manque ;
- **SELLER_REGISTRATION_VERIFIED** : l'enregistrement de l'entité vendeuse évaluée pour le champ applicable est vérifié ;
- **UPSTREAM_COVERAGE_VERIFIED** : une règle vérifiée autorise la couverture par l'entité amont nommée et le lien au produit/parcours est prouvé ;
- **NO_SELLER_REGISTRATION_REQUIRED** : la règle applicable établit qu'aucun nouvel enregistrement de l'entité vendeuse évaluée n'est requis pour ce parcours ;
- **UNRESOLVED** : les règles, le rôle, le stock, l'identité ou les preuves ne permettent pas de trancher.

OUI n'est possible que si l'état interne est VÉRIFIÉ et si l'évaluation d'inscription est résolue de façon favorable. Tous les autres états internes donnent NON. Un NON fondé sur une incertitude ne signifie pas que la vente est juridiquement interdite ni que l'obligation propre de l'entité vendeuse évaluée est établie. Aucun score de confiance visible.

Principe :
> **UNKNOWN ≠ COMPLIANT**

Une preuve insuffisante, une entité non résolue, une filière non vérifiée ou un registre indisponible ne doivent jamais produire l'état interne VÉRIFIÉ ; côté utilisateur, ils donnent NON avec une explication.

## 5. Dashboard

Afficher :
- produit ;
- EAN/GTIN ;
- ASIN si connu ;
- marque ;
- fabricant probable ;
- catégorie ;
- fournisseur et pays d'établissement ;
- pays de stock initial ;
- pays de prep center traversés ;
- pays d'inventaire FBA connus et programme FBA ;
- marché cible ;
- décision OUI / NON pour FR/DE/IT/ES/GB, une par route × marché × filière ;
- routes candidates générées à partir du profil client, avec les filtres opérationnels appliqués et les routes écartées expliquées ;
- décision d'inscription propre à l'entité vendeuse évaluée et entité responsable retenue ;
- parcours (source → stock/transit → FBA → marché) auquel chaque état s'applique ;
- obligations REP détectées ;
- entités juridiques ;
- numéros/enregistrements trouvés ;
- distinction entre l'entité vendeuse évaluée et l'entité amont prétendue responsable ;
- raison du verdict ;
- sources et preuves ;
- limites de l'analyse.

## 6. Profil utilisateur

Le profil client peut enregistrer son entité vendeuse, les pays d'immatriculation TVA, les prep centers réellement disponibles, les programmes FBA et les pays d'inventaire autorisés ou connus. L'analyse génère les routes candidates à partir de ce profil et des données produit/fournisseur ; elle ne dépend pas d'une route prédéfinie.

Les identifiants de société, les enregistrements REP/EPR, les places de marché et les paramètres FBA ne sont plus de simples données futures : ils sont des éléments du parcours à vérifier. L'intégration Amazon peut rester facultative ; les faits que le moteur ne peut pas obtenir de façon fiable doivent être saisis ou importés par l'utilisateur, sinon la décision affichée est NON et l'état interne reste NON VÉRIFIÉ.

## 7. Amazon

Le client évalué est vendeur sur Amazon ; Amazon fournit la place de marché et le fulfillment FBA selon le programme utilisé. Amazon SP-API demeure un **provider optionnel**. Le programme et les pays FBA permis/connus peuvent venir de l'API appelante, d'un import ou d'une saisie contrôlée. S'ils ne sont pas déterminables, le moteur ne doit pas les déduire du pays de TVA ou de la marketplace cible.

Le moteur ne doit pas dépendre techniquement de :
- compte Amazon Pro ;
- credentials SP-API ;
- connexion Seller Central.

Sans connexion, le MVP doit accepter les données enrichies de configuration et d'éligibilité depuis l'application appelante, ou un import contrôlé. Une connexion Amazon peut aider à obtenir les listings et données logistiques, mais ne constitue jamais une preuve juridique REP/EPR et ne doit pas servir à deviner les obligations.

## 8. Pipeline

```text
Identifier
→ Product Resolution
→ Brand Resolution
→ Entity Resolution
→ Corporate Graph
→ Seller and supplier identity + responsibility claim
→ Supplier stock → prep center(s) → FBA inventory country/countries
→ Amazon target marketplace
→ EPR Classification
→ Official Registry Matching
→ Country/Supply-chain Rules
→ États internes EPR + décision d'inscription
→ Décision utilisateur OUI / NON
→ Explication en langage clair + preuves traçables
```

## 9. Product Resolution

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

Un GTIN/ASIN ne décrit pas nécessairement les matériaux et quantités d'emballage, une batterie intégrée, un lot ou une variante de marché. La classification doit s'appuyer sur des attributs vérifiés ou une validation manuelle ; ne pas déduire une filière complète du seul identifiant.

## 10. Entity Resolution

Distinguer explicitement :
- marque ;
- propriétaire de marque ;
- titulaire GTIN ;
- fabricant ;
- parent group ;
- filiale locale ;
- distributeur ;
- importateur ;
- producteur REP.

Un même nom ne doit jamais être automatiquement assimilé à tous ces rôles.

## 11. Sources V1

Priorité aux données gratuites/officielles :
- GS1 si accès gratuit exploitable ;
- GLEIF ;
- EUIPO ;
- API Recherche Entreprises France ;
- INPI/RNE ;
- Companies House ;
- ADEME ;
- LUCID/ZSVR ;
- Stiftung EAR ;
- GOV.UK EPR/WEEE ;
- registres officiels Espagne ;
- registres officiels Italie.

## 12. Filières V1

Minimum :
- PACKAGING
- EEE/WEEE
- BATTERY

Le modèle doit rester extensible.

## 13. Registre REP

Pour chaque enregistrement conserver :
- pays ;
- filière ;
- entité ;
- registration number ;
- registre ;
- statut ;
- source ;
- date de récupération ;
- raw payload utile.

## 14. Génération des routes et règles minimales

La génération est multi-client et dynamique : à partir du profil du client, des sources/lieux de stock, du produit, des prep centers disponibles, des contraintes FBA connues et des marketplaces cibles, produire les routes candidates. Ne pas coder une route unique ni croiser arbitrairement tous les pays. Un chemin n'est candidat que si les transitions logistiques et l'éligibilité du programme FBA sont appuyées par les données ou contraintes disponibles.

Les pays où le client détient un numéro de TVA peuvent filtrer ou annoter les routes selon une règle de routage documentée. Ils ne prouvent ni l'éligibilité au stockage FBA, ni la conformité REP/EPR, ni la couverture par un tiers. Si le profil ne permet pas d'établir les pays FBA pertinents, ne pas les inventer : conserver un candidat incomplet, lui attribuer NON et l'état interne NON VÉRIFIÉ.

- calculer une ligne de résultat par combinaison prise en charge : produit × entité vendeuse × parcours physique × marché cible × filière ;
- ne pas confondre le pays d'établissement du fournisseur, le pays de son entrepôt, les pays de transit/prep center, le pays d'inventaire FBA et le pays de vente ;
- pays de stockage/inventaire FBA inconnu ou non documenté => NON côté utilisateur et NON VÉRIFIÉ en interne pour tout résultat qui dépend de cette information ;
- l'enregistrement d'une entité amont ne dispense pas automatiquement l'entité vendeuse d'une obligation ; appliquer une règle vérifiée pour le rôle et le marché concernés ;
- ne considérer une couverture amont que si l'identité légale, le statut producteur, le pays/marché, la filière/catégorie et le lien à ce produit/SKU sont établis par des preuves adaptées ;
- filière obligatoire manquante ou non classifiable => NON VÉRIFIÉ ;
- entité ou rôle dans le parcours non résolu => NON VÉRIFIÉ ;
- registre indisponible sans cache dont la fraîcheur est définie => NON VÉRIFIÉ ;
- numéro REP de la marque seul => insuffisant ;
- numéro d'un autre pays => insuffisant pour le pays cible ;
- achat dans un pays ≠ preuve de couverture dans un autre ;
- produit multi-filières : toutes les filières requises doivent être vérifiées.
- absence de correspondance dans un registre => NON CONFORME uniquement si la source est complète, à jour selon une règle documentée, et l'entité est résolue sans ambiguïté ; sinon NON VÉRIFIÉ.
- un IDU établit l'enregistrement de l'entité pour une filière et des catégories données ; il ne relie pas à lui seul l'entité au GTIN, au vendeur, au contrat ou au parcours analysé.

## 15. Reason Codes

Prévoir au minimum :
- ALL_REQUIRED_STREAMS_VERIFIED
- LOCAL_RESPONSIBLE_ENTITY_VERIFIED
- NO_LOCAL_ENTITY_FOUND
- EPR_REGISTRATION_NOT_FOUND
- MULTIPLE_STREAMS_INCOMPLETE
- CROSS_BORDER_COVERAGE_NOT_ESTABLISHED
- PRODUCT_CLASSIFICATION_UNCERTAIN
- ENTITY_RESOLUTION_FAILED
- OFFICIAL_REGISTRY_UNAVAILABLE
- INSUFFICIENT_OFFICIAL_EVIDENCE
- PRODUCT_NOT_RESOLVED
- SELLER_REGISTRATION_REQUIRED_MISSING
- SELLER_REGISTRATION_VERIFIED
- UPSTREAM_COVERAGE_VERIFIED
- UPSTREAM_COVERAGE_UNPROVEN
- SELLER_RESPONSIBILITY_UNRESOLVED
- SELLER_ENTITY_NOT_RESOLVED
- SUPPLIER_ENTITY_NOT_RESOLVED
- UPSTREAM_COVERAGE_NOT_LINKED_TO_PRODUCT
- FBA_INVENTORY_COUNTRY_UNKNOWN
- FULFILLMENT_PROGRAM_UNKNOWN
- ROUTE_NOT_SUPPORTED
- MARKET_RULE_NOT_VALIDATED
- REGISTRATION_SCOPE_MISMATCH

## 16. API interne

Le dashboard et toute application cliente tierce utilisent le même moteur et le même contrat API. L'identité du client/tenant provient du contexte d'authentification, jamais d'un `tenant_id` fourni sans contrôle dans le corps de la requête.

### POST /api/analyses

```json
{
  "identifier": "8710103995426",
  "seller": {
    "legal_entity_id": "...",
    "country_of_establishment": "...",
    "existing_epr_evidence_ids": ["..."]
  },
  "client_configuration": {
    "vat_registration_countries": ["FR", "DE"],
    "available_prep_centers": [
      {"id": "prep-1", "country": "FR", "eligible_product_types": ["..."]}
    ],
    "amazon_fulfillment_profiles": [
      {"program": "...", "eligible_inventory_countries": ["FR", "DE"]}
    ],
    "supported_logistics_transitions": [
      {"from_stock_country": "DE", "via_prep_center_id": "prep-1", "to_fba_country": "FR"}
    ]
  },
  "sourcing_options": [
    {
      "supplier_entity_id": "...",
      "supplier_establishment_country": "DE",
      "available_stock_countries": ["DE", "PL"]
    }
  ],
  "target_markets": ["FR", "DE", "IT", "ES", "GB"],
  "product_facts": {
    "category_attributes": {},
    "packaging_attributes": {},
    "embedded_battery": null
  },
  "upstream_claims": [
    {
      "claimed_responsible_entity_id": "...",
      "product_or_sku_scope": "...",
      "stream": "EEE",
      "evidence_ids": ["..."]
    }
  ]
}
```

Réponse :
```json
{
  "analysis_id": "uuid",
  "product": {},
  "route_generation": {
    "generated_routes_count": 1,
    "excluded_combinations": [
      {
        "source_option_id": "...",
        "target_market": "IT",
        "explanation": "Aucune transition logistique ou éligibilité FBA suffisamment étayée."
      }
    ]
  },
  "routes": [
    {
      "route_id": "...",
      "supplier_establishment_country": "DE",
      "supplier_stock_country": "DE",
      "prep_center_countries": ["FR"],
      "amazon_fulfillment_program": "...",
      "fba_inventory_countries": ["FR"],
      "target_market": "FR",
      "stream": "EEE",
      "decision": "NON",
      "explanation": "Les preuves disponibles ne démontrent pas que la route peut être poursuivie sans nouvel enregistrement.",
      "internal": {
        "epr_status": "UNVERIFIED",
        "registration_assessment": {
          "outcome": "UNRESOLVED",
          "entity_to_register": null,
          "responsible_entity_id": null,
          "evidence_ids": []
        },
        "action": "HOLD",
        "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE"
      },
      "scope": {
        "seller_entity_id": "...",
        "claimed_responsible_entity_id": "...",
        "rule_version": "..."
      }
    }
  ]
}
```

Le moteur génère les routes candidates à partir des transitions logistiques fournies ou vérifiées, sans omettre silencieusement un marché. Chaque résultat porte sur un parcours, un marché et une filière. L'interface n'affiche que `decision: OUI | NON` ; les champs `internal` ne servent qu'à l'évaluation, à l'audit et au calcul. Le schéma doit versionner les règles appliquées et conserver la date de l'analyse ainsi que les dates de récupération des preuves. `action` est un garde-fou interne au périmètre REP/EPR évalué, pas une autorisation générale de mise en vente.

Prévoir aussi :
- GET /api/analyses/:id
- GET /api/analyses
- GET /api/products/:id

## 17. Données

PostgreSQL + Prisma.

Tables :
- users
- user_prep_centers
- products
- product_identifiers
- supply_routes
- route_segments
- seller_profiles
- supplier_profiles
- brands
- entities
- entity_aliases
- entity_relationships
- brand_entity_relationships
- epr_streams
- epr_registrations
- product_epr_classifications
- analyses
- route_market_stream_results
- registration_assessments
- evidence

## 18. Providers

Créer une abstraction homogène.

```typescript
interface DataProvider {
  search(params: unknown): Promise<ProviderResult>;
}

interface EprRegistryProvider {
  findRegistration(
    entity: Entity,
    stream: EprStream,
    category: EprCategory,
    country: CountryCode
  ): Promise<EprRegistryResult>;
}
```

Toute clé facultative absente doit produire un état provider contrôlé, jamais empêcher le démarrage de l'application.

## 19. Cache & synchronisation

Ne pas interroger chaque registre à chaque analyse.

Synchroniser les jeux bulk lorsque possible et stocker localement.

Caches :
- identifier → product ;
- brand → entity graph ;
- entity + country + stream + category → registration ;
- seller + product + full route + target market + stream + rule version → analysis result.

Un changement de pays FBA, de prep center, de marché cible, d'entité responsable ou de programme de fulfillment invalide le résultat correspondant et exige une nouvelle analyse.

## 20. Sécurité

- secrets en variables d'environnement ;
- aucune clé côté frontend ;
- validation stricte des inputs ;
- rate limiting ;
- logs sans secrets ;
- sanitation des réponses externes ;
- auth utilisateur.

## 21. Tests

Unitaires :
- détection EAN/GTIN/UPC/ASIN ;
- normalisation d'entreprise ;
- classification filières ;
- moteur d'états VÉRIFIÉ / NON CONFORME / NON VÉRIFIÉ / CONTRADICTOIRE ;
- données manquantes/contradictoires.

Intégration :
- chaque provider.

E2E :
- emballage seul ;
- EEE+batterie ;
- produit introuvable ;
- société non résolue ;
- numéro REP trouvé mais relation insuffisante ;
- registre indisponible ;
- correspondance d'entité ambiguë ;
- absence d'enregistrement avec source incomplète ou périmée ;
- conflit de sources ;
- preuve positive sur le producteur, mais lien au vendeur/parcours non établi ;
- même produit de DE vers FR via un prep center FR, avec des pays FBA/marketplace distincts ;
- plusieurs pays de stock FBA connus pour un même produit ;
- pays d'inventaire FBA inconnu ;
- preuve amont existante mais catégorie, SKU ou lien au contrat non vérifié ;
- route ou règle nationale non couverte.

Prévoir un jeu de référence relu indépendamment et mesurer au minimum la fiabilité des états VÉRIFIÉ, la part des cas vérifiables et les parts NON VÉRIFIÉ / CONTRADICTOIRE. Fixer les seuils de lancement avant d'évaluer le jeu, puis mesurer sur des cas tenus à l'écart de l'élaboration des règles. Un taux de faux positifs nul sur un petit jeu ne constitue pas une garantie universelle.

## 22. Stack

- Next.js
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- PostgreSQL
- Prisma
- Zod
- Vitest
- Playwright

Pas de microservices en V1.

## 23. Hors périmètre V1

Ne pas implémenter :
- plateforme de sourcing ;
- comparaison de fournisseurs ;
- ROI ;
- Keepa ;
- marge FBA ;
- inscription REP automatisée ;
- fiscalité complète ;
- APIs corporate payantes obligatoires ;
- avis juridique définitif.

## 24. Critères d'acceptation

Le pilote est fonctionnel si :
1. un EAN/GTIN/ASIN et ses attributs produit vérifiés peuvent être fournis ;
2. les routes candidates sont générées depuis les options source/stock, le profil client, les prep centers, les transitions logistiques, la configuration FBA et les marchés demandés — aucune route n'est codée en dur ;
3. le moteur fonctionne pour plusieurs clients et isole leurs profils, preuves et résultats ;
4. le dashboard et une application tierce peuvent appeler le même contrat API avec des données enrichies ;
5. chaque route générée distingue fournisseur, stock initial, prep centers, pays/programme FBA et marché cible ;
6. chaque combinaison demandée de route × marché × filière reçoit une décision OUI/NON ; un parcours ou une règle non pris en charge donne NON côté utilisateur et conserve son état NON VÉRIFIÉ en interne ;
7. les obligations propres à l'entité vendeuse du client sont distinguées d'une éventuelle couverture par une entité amont ;
8. chaque route indique en interne si la société doit obtenir un enregistrement, si son propre enregistrement est vérifié, si la couverture amont est prouvée, ou si la réponse reste indéterminée ;
9. tout résultat UPSTREAM_COVERAGE_VERIFIED repose sur une règle pays/acteur/filière validée et une preuve reliant l'entité enregistrée au produit/SKU et au parcours ;
10. les états techniques sont distingués en interne, mais ne produisent jamais une troisième décision visible au-delà de OUI/NON ;
11. chaque preuve est datée, attribuée à une source et reliée à sa portée ;
12. le jeu de référence permet d'évaluer fiabilité positive et couverture selon des seuils décidés à l'avance ;
13. le résultat est présenté comme un contrôle REP/EPR borné et non comme un avis juridique ou une autorisation générale de vente ;
14. le MVP ne dépend ni d'une API commerciale payante ni d'une connexion Amazon obligatoire.

La matrice de routes peut couvrir les cinq marchés cibles déclarés et les pays d'approvisionnement explicitement pris en charge ; un verdict VÉRIFIÉ n'est disponible que pour les combinaisons dont les règles et sources ont été auditées.
