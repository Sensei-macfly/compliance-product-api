# Cahier des charges V1 — Compliance Product API

## 1. Objectif

Construire un MVP autonome qui vérifie, pour un parcours REP/EPR précisément défini, les obligations et preuves identifiables à partir de sources auditables. La V1 ne doit pas prétendre autoriser globalement la vente d'un produit.

> **Pour cette route et ce marché, peut-on poursuivre sans nouvel enregistrement REP/EPR/IDU de notre société, preuves à l'appui ?**

Le pilote sert à valider une brique qui pourra ensuite être intégrée à Solenda. Il ne doit pas devenir une plateforme de sourcing. La question de revente ci-dessus reste la vision produit à long terme, pas une promesse du premier pilote.

## 2. Entrées

Le modèle commercial confirmé est le suivant : notre société est vendeuse sur Amazon ; elle achète auprès de fournisseurs/grossistes situés en France, en Allemagne, en Italie, en Espagne et ailleurs dans l'Union européenne ; les produits peuvent transiter par un prep center dans un autre pays avant Amazon FBA, puis être vendus sur plusieurs marchés. Ce contexte doit être évalué par parcours réel, pas résumé par un pays de stock unique.

Deux champs avaient été proposés pour l'expérience initiale, mais ils ne suffisent pas à établir à qui incombe l'obligation REP/EPR :
1. EAN / GTIN / UPC / ASIN, comme aide à l'identification — pas comme preuve de catégorie ou de couverture ;
2. pays où le produit est physiquement stocké chez le fournisseur, comme contexte logistique — pas comme preuve du rôle REP de chaque acteur.

Chaque analyse doit distinguer et conserver :
- l'entité juridique et le pays d'établissement de notre société vendeuse ;
- les enregistrements REP/EPR déjà détenus par notre société et leurs pays/filières/catégories ;
- le fournisseur/grossiste et le pays où il est établi ;
- le lieu de stockage physique chez le fournisseur, distinct de son pays d'établissement ;
- chaque pays de prep center traversé ;
- le ou les pays où l'inventaire Amazon FBA est effectivement stocké, ainsi que le programme d'expédition/stockage FBA utilisé ;
- le marché Amazon cible où la vente est réalisée ;
- le fabricant/producteur ou l'autre entité amont qui est prétendue responsable REP/EPR, son identifiant juridique et ses enregistrements ;
- les pièces reliant cette entité à notre société, au produit/SKU, à la filière/catégorie et au marché concerné.

L'identifiant produit (EAN/GTIN/ASIN) et les attributs nécessaires à la classification restent requis, mais ne remplacent aucun de ces faits de parcours. Une absence d'information pertinente produit la décision utilisateur **NON** ; en interne, elle reste NON VÉRIFIÉE, sans déduction à partir du pays du fournisseur.

Le pays d'établissement du fournisseur, le pays de stock initial, les pays de transit/stockage et le marché cible sont des axes distincts. Ne pas supposer qu'un prep center ou une immatriculation amont transfère automatiquement la couverture REP/EPR. Ne pas demander un numéro de TVA par principe : demander l'identifiant légal réellement nécessaire à la résolution de chaque entité dans la source utilisée. Pour les entités étrangères, la source ADEME peut utiliser un identifiant fiscal étranger.

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

Automatiquement :
- FR
- DE
- IT
- ES
- GB

FR, DE, IT, ES et GB restent les marchés cibles déclarés ; les pays fournisseurs/stock peuvent inclure d'autres pays de l'UE. Produire une analyse pour chaque combinaison de parcours d'approvisionnement/stockage pris en charge et marché cible demandé. Les pays ou filières sans règles et preuves auditées retournent **NON** à l'utilisateur, jamais un OUI implicite. Ajouter d'autres marchés de vente exige une décision de périmètre.

La source ADEME France / EEE est un candidat pour vérifier une pièce d'enregistrement française. Elle ne valide pas à elle seule un parcours Amazon FBA transfrontalier, la responsabilité de la société vendeuse ou une couverture dans les autres pays.

## 4. Verdict

Les états ci-dessous sont **internes** ; ils ne constituent pas des choix supplémentaires dans l'interface utilisateur :
- **VÉRIFIÉ** : les preuves requises pour le parcours REP/EPR défini sont suffisantes et cohérentes ;
- **NON CONFORME** : une obligation applicable est établie et des preuves suffisantes montrent qu'elle n'est pas remplie ;
- **NON VÉRIFIÉ** : informations, correspondance d'entité, couverture de source ou preuves insuffisantes ;
- **CONTRADICTOIRE** : des sources pertinentes se contredisent.

L'interface n'affiche que **OUI / NON** à la question « peut-on poursuivre cette route sans nouvel enregistrement de notre société ? ». OUI exige des preuves suffisantes et cohérentes. NON couvre tous les autres cas, y compris les preuves manquantes ou contradictoires. L'explication peut décrire le blocage, mais ne présente jamais NON VÉRIFIÉ comme une troisième décision commerciale.

En interne, chaque résultat précise la situation d'inscription, sans déduire une dispense de la seule présence d'un IDU amont :
- **SELLER_REGISTRATION_MISSING** : une règle applicable établit que notre société doit être enregistrée, mais la preuve d'un enregistrement valable manque ;
- **SELLER_REGISTRATION_VERIFIED** : l'enregistrement de notre société pour le champ applicable est vérifié ;
- **UPSTREAM_COVERAGE_VERIFIED** : une règle vérifiée autorise la couverture par l'entité amont nommée et le lien au produit/parcours est prouvé ;
- **NO_SELLER_REGISTRATION_REQUIRED** : la règle applicable établit qu'aucun nouvel enregistrement de notre société n'est requis pour ce parcours ;
- **UNRESOLVED** : les règles, le rôle, le stock, l'identité ou les preuves ne permettent pas de trancher.

OUI n'est possible que si l'état interne est VÉRIFIÉ et si l'évaluation d'inscription est résolue de façon favorable. Tous les autres états internes donnent NON. Un NON fondé sur une incertitude ne signifie pas que la vente est juridiquement interdite ni que l'obligation propre de notre société est établie. Aucun score de confiance visible.

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
- décision d'inscription propre à notre société et entité responsable retenue ;
- parcours (source → stock/transit → FBA → marché) auquel chaque état s'applique ;
- obligations REP détectées ;
- entités juridiques ;
- numéros/enregistrements trouvés ;
- distinction entre l'entité vendeuse évaluée et l'entité amont prétendue responsable ;
- raison du verdict ;
- sources et preuves ;
- limites de l'analyse.

## 6. Profil utilisateur

Le profil peut enregistrer les entités juridiques de la société vendeuse et les prep centers, mais l'analyse doit permettre de choisir les pays/parcours réellement utilisés pour le produit.

Les identifiants de société, les enregistrements REP/EPR, les places de marché et les paramètres FBA ne sont plus de simples données futures : ils sont des éléments du parcours à vérifier. L'intégration Amazon peut rester facultative ; les faits que le moteur ne peut pas obtenir de façon fiable doivent être saisis ou importés par l'utilisateur, sinon la décision affichée est NON et l'état interne reste NON VÉRIFIÉ.

## 7. Amazon

Notre société est le vendeur sur Amazon ; Amazon fournit la place de marché et le fulfillment FBA selon le programme utilisé. Amazon SP-API demeure un **provider optionnel**, mais les marchés de vente, le programme FBA et les pays d'inventaire connus doivent être fournis ou importés pour chaque parcours.

Le moteur ne doit pas dépendre techniquement de :
- compte Amazon Pro ;
- credentials SP-API ;
- connexion Seller Central.

Sans connexion, le MVP doit accepter une saisie ou un import contrôlé des routes et lieux de stock FBA. Une connexion Amazon peut aider à obtenir les listings et données logistiques, mais ne constitue jamais une preuve juridique REP/EPR et ne doit pas servir à deviner les obligations.

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

## 14. Règles minimales

- calculer une ligne de résultat par combinaison prise en charge : produit × entité vendeuse × parcours physique × marché cible × filière ;
- ne pas confondre le pays d'établissement du fournisseur, le pays de son entrepôt, les pays de transit/prep center, le pays d'inventaire FBA et le pays de vente ;
- pays de stockage/inventaire FBA inconnu ou non documenté => NON VÉRIFIÉ pour tout résultat qui dépend de cette information ;
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

### POST /api/analyses

```json
{
  "identifier": "8710103995426",
  "seller": {
    "legal_entity_id": "...",
    "country_of_establishment": "...",
    "existing_epr_evidence_ids": ["..."]
  },
  "sales_channel": {
    "marketplace": "AMAZON",
    "fulfillment": "FBA"
  },
  "product_facts": {
    "category_attributes": {},
    "packaging_attributes": {},
    "embedded_battery": null
  },
  "routes": [
    {
      "supplier_entity_id": "...",
      "supplier_establishment_country": "DE",
      "supplier_stock_country": "DE",
      "prep_center_countries": ["FR"],
      "amazon_fulfillment_program": "...",
      "fba_inventory_countries": ["FR"],
      "target_markets": ["FR", "DE", "IT", "ES", "GB"],
      "claimed_responsible_entity_id": "...",
      "upstream_evidence_ids": ["..."]
    }
  ]
}
```

Réponse :
```json
{
  "analysis_id": "uuid",
  "product": {},
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

Le moteur doit retourner toutes les combinaisons demandées dans le périmètre déclaré, sans omettre silencieusement un trajet. Chaque résultat porte sur un parcours, un marché et une filière. L'interface n'affiche que `decision: OUI | NON` ; les champs `internal` ne servent qu'à l'évaluation, à l'audit et au calcul. Le schéma doit versionner les règles appliquées et conserver la date de l'analyse ainsi que les dates de récupération des preuves. `action` est un garde-fou interne au périmètre REP/EPR évalué, pas une autorisation générale de mise en vente.

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
1. un EAN/GTIN/ASIN peut être saisi ;
2. chaque route décrit séparément le pays du fournisseur, le stock initial, les prep centers, le ou les pays d'inventaire FBA et le marché cible ;
3. chaque combinaison demandée de route × marché × filière reçoit une décision OUI/NON ; un parcours ou une règle non pris en charge donne NON côté utilisateur et conserve son état NON VÉRIFIÉ en interne ;
4. les obligations propres à la société vendeuse sont distinguées d'une éventuelle couverture par une entité amont ;
5. chaque route indique si la société doit obtenir un enregistrement, si son propre enregistrement est vérifié, si la couverture amont est prouvée, ou si la réponse reste indéterminée ;
6. tout résultat UPSTREAM_COVERAGE_VERIFIED repose sur une règle pays/acteur/filière validée et une preuve reliant l'entité enregistrée au produit/SKU et au parcours ;
7. les états techniques sont distingués en interne, mais ne produisent jamais une troisième décision visible au-delà de OUI/NON ;
8. chaque preuve est datée, attribuée à une source et reliée à sa portée ;
9. le jeu de référence permet d'évaluer fiabilité positive et couverture selon des seuils décidés à l'avance ;
10. le résultat est présenté comme un contrôle REP/EPR borné et non comme un avis juridique ou une autorisation générale de vente ;
11. le MVP ne dépend ni d'une API commerciale payante ni d'une connexion Amazon obligatoire ;
12. le contrat API permet une future intégration à Solenda après validation.

La matrice de routes peut couvrir les cinq marchés cibles déclarés et les pays d'approvisionnement explicitement pris en charge ; un verdict VÉRIFIÉ n'est disponible que pour les combinaisons dont les règles et sources ont été auditées.
