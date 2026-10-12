# Cahier des charges V1 — Compliance Product API

## 1. Objectif

Construire un MVP autonome de pré-validation REP/EPR permettant de répondre :

> **Ce produit est stocké chez mon fournisseur dans ce pays. Dans quels marchés puis-je le revendre avec mon modèle actuel, sans nouvelle immatriculation REP/EPR/IDU ?**

Le MVP sert à valider une brique qui pourra ensuite être intégrée à Solenda. Il ne doit pas devenir une plateforme de sourcing.

## 2. Entrées

Deux champs obligatoires :
1. EAN / GTIN / UPC / ASIN ;
2. pays où le produit est physiquement stocké chez le fournisseur.

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

## 4. Verdict

Décision utilisateur uniquement :
- **OUI**
- **NON**

Pas de score visible.

Principe :
> **UNKNOWN ≠ COMPLIANT**

Une preuve insuffisante, une entité non résolue, une filière non vérifiée ou un registre indisponible ne doivent jamais produire OUI.

## 5. Dashboard

Afficher :
- produit ;
- EAN/GTIN ;
- ASIN si connu ;
- marque ;
- fabricant probable ;
- catégorie ;
- pays stock fournisseur ;
- verdict OUI/NON pour FR/DE/IT/ES/GB ;
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
→ Local Entity FR/DE/IT/ES/GB
→ EPR Classification
→ Official Registry Matching
→ Country/Supply-chain Rules
→ OUI/NON
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

- filière obligatoire manquante => NON ;
- entité locale non résolue => NON ;
- registre indisponible sans cache récent => NON ;
- numéro REP de la marque seul => insuffisant ;
- numéro d'un autre pays => insuffisant pour le pays cible ;
- achat dans un pays ≠ preuve de couverture dans un autre ;
- produit multi-filières : toutes les filières requises doivent être vérifiées.

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
    "FR": {"verdict": "NO"},
    "DE": {"verdict": "YES"},
    "IT": {"verdict": "NO"},
    "ES": {"verdict": "YES"},
    "GB": {"verdict": "NO"}
  }
}
```

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
- moteur OUI/NON ;
- données manquantes/contradictoires.

Intégration :
- chaque provider.

E2E :
- emballage seul ;
- EEE+batterie ;
- produit introuvable ;
- société non résolue ;
- numéro REP trouvé mais relation insuffisante ;
- registre indisponible.

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

La V1 est fonctionnelle si :
1. un EAN/GTIN/ASIN peut être saisi ;
2. le pays stock fournisseur peut être sélectionné ;
3. le produit est résolu autant que possible ;
4. les filières REP sont classifiées ;
5. les entités sont résolues ;
6. les registres officiels sont interrogés/synchronisés ;
7. FR/DE/IT/ES/GB reçoivent chacun OUI/NON ;
8. chaque verdict possède une raison ;
9. chaque preuve possède une source ;
10. une donnée insuffisante donne NON ;
11. aucune API commerciale payante n'est indispensable ;
12. aucune connexion Amazon n'est indispensable ;
13. le moteur est consommable via API pour une future intégration Solenda.
