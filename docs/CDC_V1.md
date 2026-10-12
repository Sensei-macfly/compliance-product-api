# Cahier des charges V1 — Compliance Product API

## 1. Objectif

Construire un MVP autonome qui vérifie, pour un parcours REP/EPR précisément défini, les obligations et preuves identifiables à partir de sources auditables. La V1 ne doit pas prétendre autoriser globalement la vente d'un produit.

> **Ce produit est stocké chez mon fournisseur dans ce pays. Dans quels marchés puis-je le revendre avec mon modèle actuel, sans nouvelle immatriculation REP/EPR/IDU ?**

Le pilote sert à valider une brique qui pourra ensuite être intégrée à Solenda. Il ne doit pas devenir une plateforme de sourcing. La question de revente ci-dessus reste la vision produit à long terme, pas une promesse du premier pilote.

## 2. Entrées

Deux champs ont été proposés pour l'expérience initiale, mais ils ne suffisent pas à établir à qui incombe l'obligation REP/EPR :
1. EAN / GTIN / UPC / ASIN, comme aide à l'identification — pas comme preuve de catégorie ou de couverture ;
2. pays où le produit est physiquement stocké chez le fournisseur, comme contexte logistique — pas comme preuve du rôle REP de chaque acteur.

Avant de figer l'UX, le pilote doit déterminer les informations minimales nécessaires au cas analysé :
- l'entité juridique dont on évalue les obligations, identifiée sans ambiguïté ;
- son rôle dans le parcours (vendeur, importateur, metteur sur le marché, place de marché, etc.) et le canal de vente retenu ;
- l'entité qui déclare remplir les obligations pour le produit et les preuves qui relient cette entité au produit/parcours ;
- les caractéristiques produit nécessaires à la classification de la filière et, le cas échéant, de ses catégories.

Le pays de stock reste une donnée de contexte tant qu'une règle vérifiée ne démontre pas son effet sur le parcours retenu. Ne pas demander un numéro de TVA par principe : demander l'identifiant légal réellement nécessaire à la résolution de l'entité dans la source utilisée. Pour les entités étrangères, la source ADEME peut utiliser un identifiant fiscal étranger.

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

Ce périmètre est une cible ultérieure, pas le périmètre du premier pilote. Le pilote de faisabilité de source proposé est France / filière EEE ; il vérifie une preuve d'enregistrement d'une entité et d'une filière. Il ne démontre pas à lui seul qu'un produit particulier peut être vendu sans autre démarche.

## 4. Verdict

Ne pas confondre état de la preuve et décision de prudence :
- **VÉRIFIÉ** : les preuves requises pour le parcours REP/EPR défini sont suffisantes et cohérentes ;
- **NON CONFORME** : une obligation applicable est établie et des preuves suffisantes montrent qu'elle n'est pas remplie ;
- **NON VÉRIFIÉ** : informations, correspondance d'entité, couverture de source ou preuves insuffisantes ;
- **CONTRADICTOIRE** : des sources pertinentes se contredisent.

La recommandation opérationnelle peut rester binaire : poursuivre le parcours REP/EPR uniquement si l'état est VÉRIFIÉ ; sinon, suspendre et demander une vérification. « Suspendre » ou « NON VÉRIFIÉ » ne signifie pas que la vente est juridiquement interdite. Aucun score de confiance visible.

Principe :
> **UNKNOWN ≠ COMPLIANT**

Une preuve insuffisante, une entité non résolue, une filière non vérifiée ou un registre indisponible ne doivent jamais produire l'état VÉRIFIÉ.

## 5. Dashboard

Afficher :
- produit ;
- EAN/GTIN ;
- ASIN si connu ;
- marque ;
- fabricant probable ;
- catégorie ;
- pays stock fournisseur ;
- état VÉRIFIÉ / NON CONFORME / NON VÉRIFIÉ / CONTRADICTOIRE pour FR/DE/IT/ES/GB ;
- obligations REP détectées ;
- entités juridiques ;
- numéros/enregistrements trouvés ;
- raison du verdict ;
- sources et preuves ;
- limites de l'analyse.

## 6. Profil utilisateur

Facultatif en V1 :
- prep centers disponibles.

Prévoir la structure pour stocker ultérieurement :
- TVA ;
- numéros REP ;
- marketplaces ;
- paramètres FBA.

Aucune de ces informations ne doit bloquer la V1.

## 7. Amazon

Amazon SP-API est un **provider optionnel**.

Le MVP doit fonctionner sans :
- compte Amazon Pro ;
- credentials SP-API ;
- connexion Seller Central.

Si connectée, Amazon enrichit l'identification produit mais ne constitue jamais une preuve juridique REP.

## 8. Pipeline

```text
Identifier
→ Product Resolution
→ Brand Resolution
→ Entity Resolution
→ Corporate Graph
→ Seller / responsible-entity + sales-path resolution
→ Local Entity FR/DE/IT/ES/GB
→ EPR Classification
→ Official Registry Matching
→ Country/Supply-chain Rules
→ EPR_STATUS + action
→ Evidence + Reason Codes
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

## 16. API interne

### POST /api/analyses

```json
{
  "identifier": "8710103995426",
  "supplier_stock_country": "DE"
}
```

Réponse :
```json
{
  "analysis_id": "uuid",
  "product": {},
  "markets": {
    "FR": {
      "epr_status": "UNVERIFIED",
      "action": "HOLD",
      "reason_code": "INSUFFICIENT_OFFICIAL_EVIDENCE",
      "scope": {
        "stream": "EEE",
        "rule_version": "..."
      }
    }
  }
}
```

Le schéma doit aussi versionner les règles appliquées et conserver la date de l'analyse ainsi que les dates de récupération des preuves. `action` est un garde-fou pour le périmètre REP/EPR évalué, pas une autorisation générale de mise en vente.

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
- brands
- entities
- entity_aliases
- entity_relationships
- brand_entity_relationships
- epr_streams
- epr_registrations
- product_epr_classifications
- analyses
- country_analysis_results
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
- brand + country → local entity ;
- entity + country + stream → registration.

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
- preuve positive sur le producteur, mais lien au vendeur/parcours non établi.

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
2. l'acteur, le parcours de vente et les caractéristiques nécessaires à la classification sont définis ;
3. le pilote France / EEE consulte ou synchronise la source ADEME avec des limites et dates de fraîcheur explicites ;
4. le lien entre l'entité enregistrée et l'obligation évaluée est justifié, ou l'analyse reste NON VÉRIFIÉE ;
5. les états VÉRIFIÉ / NON CONFORME / NON VÉRIFIÉ / CONTRADICTOIRE sont distingués et expliqués ;
6. chaque preuve est datée, attribuée à une source et reliée à sa portée ;
7. le jeu de référence permet d'évaluer fiabilité positive et couverture selon des seuils décidés à l'avance ;
8. le résultat est présenté comme un contrôle REP/EPR borné et non comme un avis juridique ou une autorisation générale de vente.
9. le pilote ne dépend ni d'une API commerciale payante ni d'une connexion Amazon ;
10. le contrat API reste documenté pour permettre une future intégration à Solenda après validation du pilote.

L'extension aux cinq marchés, aux trois filières et à une API intégrable à Solenda vient après la validation de ce pilote.
