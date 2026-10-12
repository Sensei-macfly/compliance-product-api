# Collaboration & source de vérité

## Objectif

Éviter que les décisions produit, réglementaires ou d'architecture divergent entre les conversations ChatGPT, Replit et le code.

## Source de vérité

La branche `main` de ce repository est la **source de vérité du projet**.

Avant toute nouvelle phase de travail, tout agent ou contributeur doit :
1. synchroniser son workspace avec `main` ;
2. lire les documents de référence ;
3. ne pas contredire une décision existante sans la signaler explicitement.

Documents de référence :
- `docs/CDC_V1.md` : périmètre fonctionnel ;
- `docs/ARCHITECTURE.md` : architecture ;
- `docs/DECISIONS.md` : décisions produit ;
- `docs/SOURCES_APIS.md` : sources et providers ;
- `docs/ROADMAP.md` : séquencement ;
- `.agents/memory/*` : contexte opérationnel destiné aux agents.

## Statut des idées

Une discussion dans ChatGPT ou Replit n'est **pas** une décision du projet tant qu'elle n'est pas intégrée dans les documents de référence.

Utiliser cette distinction :
- **DISCUSSION** : hypothèse, idée, piste, question ouverte ;
- **PROPOSAL** : changement proposé, à challenger ;
- **DECISION** : choix validé et inscrit dans les docs ;
- **IMPLEMENTED** : choix effectivement implémenté et testé.

## Workflow recommandé

```text
Discussion / analyse
        ↓
Proposition cohérente
        ↓
Branche dédiée
        ↓
Mise à jour docs et/ou code
        ↓
Pull Request
        ↓
Revue / challenge
        ↓
Merge dans main
        ↓
Tous les agents repartent de main
```

Il n'est pas nécessaire de créer une PR à chaque échange. Regrouper les changements en lots cohérents.

## Règle pour Replit

Avant de coder ou de modifier l'architecture :
- lire `main` ;
- signaler toute contradiction trouvée entre la demande courante et les docs ;
- ne pas transformer une réflexion locale en décision de projet sans mise à jour documentaire ;
- ne pas réintroduire une décision explicitement abandonnée ;
- proposer les changements structurants avant de les implémenter lorsque le périmètre est encore en discussion.

Après un travail significatif :
- résumer ce qui a changé ;
- lister les fichiers touchés ;
- préciser les hypothèses restantes ;
- pousser sur une branche dédiée ;
- ne pas merger silencieusement un changement de doctrine produit.

## Règle pour ChatGPT

Lors d'une revue :
- lire l'état actuel du repository avant de challenger ;
- considérer `main` comme la référence validée ;
- distinguer erreur, divergence, simplification et nouvelle proposition ;
- modifier le repo via une branche et une PR pour tout changement structurant.

## Rôle du propriétaire du projet

Le propriétaire tranche les désaccords de produit ou de risque réglementaire.

Il n'a pas besoin de servir de messager entre les agents : les décisions stabilisées doivent être inscrites dans GitHub afin que chacun puisse les relire.

## Discipline de branche

- `main` : état validé ;
- une branche par thème cohérent ;
- ne pas mélanger documentation, expérimentation sans rapport et code applicatif dans une même PR ;
- repartir de `main` après chaque merge avant un nouveau chantier.

## Règle de sécurité réglementaire

Aucune modification ne doit affaiblir le principe :

> UNKNOWN ≠ COMPLIANT

Toute conclusion positive doit rester traçable, expliquée et fondée sur des règles et preuves suffisamment auditées.
