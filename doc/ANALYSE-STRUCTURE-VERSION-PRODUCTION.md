# Analyse structure, versions et préparation production

## Diagnostic global

Le dépôt est un monolithe modulaire pertinent pour le MVP : `frontend/` contient l’interface React, `backend/` l’API Django, `doc/` la documentation métier et UML, et `docker-compose.yml` orchestre PostgreSQL, Redis et les deux applications. Le découpage Django par domaines (`accounts`, `equipment`, `failures`, `interventions`, `preventive`, etc.) est sain et correspond aux règles du projet.

La base est donc correcte pour le développement. Elle n’est toutefois pas prête à être déployée en production sans corrections : la séparation Domain/Application/Infrastructure reste peu matérialisée, la livraison Git est incomplète, et la configuration Docker/CI est orientée développement.

## Structure : état et améliorations

| Zone | État observé | Amélioration attendue |
| --- | --- | --- |
| `backend/apps/` | Modules métier Django distincts ; migrations par application. | Mettre les règles de transition et RB-* dans `backend/domain/`, sous forme de services/entités testables sans vues ni serializers Django. Les vues restent une couche API mince. |
| `backend/domain/` et `infrastructure/` | Arborescence créée mais majoritairement vide. | Déplacer progressivement les politiques de criticité, panne, maintenance et audit dans ces couches ; injecter les dépôts/adaptateurs depuis l’application. |
| `frontend/src/` | Pages, composants, services et types sont séparés ; `features/` existe aussi. | Choisir une convention unique : soit organisation par fonctionnalité (`features/pannes`, `features/equipment`), soit par type. Éviter de répartir une même fonctionnalité entre les deux. |
| `doc/` | Documentation métier riche et maquettes disponibles. | Ajouter un `README.md` racine : démarrage, architecture, variables, commandes et liens vers les règles RB-*. Maintenir les diagrammes PlantUML avec leur source `.puml`. |
| Données locales | `backend/db.sqlite3` et `frontend/tsconfig.tsbuildinfo` sont suivis. | Les retirer du suivi dans un commit dédié et les ajouter à `.gitignore`; conserver uniquement migrations, code et fichiers de verrouillage (`package-lock.json`). Ne supprimer aucun fichier sans validation de l’équipe. |

## Gestion des versions et Git

L’historique disponible est très court (trois commits) et les messages actuels sont vagues, par exemple `add docs`. De nombreux fichiers sont modifiés ou non suivis : une fonctionnalité peut alors être localement opérationnelle mais absente du commit, ce qui explique les erreurs de revue.

Adopter ces règles :

1. Créer une branche par sujet : `feat/synchronisation-hors-ligne`, `fix/cache-utilisateur` ou `docs/guide-production`.
2. Utiliser Conventional Commits en français ou anglais, de manière constante : `feat(sync): ajoute la réconciliation des pannes temporaires`, `fix(api): préserve la clé d'idempotence`, `docs: précise le déploiement`.
3. Garder un commit atomique : code, tests, migration Django et documentation nécessaires sont livrés ensemble. Vérifier avec `git status`, puis `git diff --check` avant le commit.
4. Protéger `main` : aucune poussée directe ; une pull request requiert revue, CI verte et résolution des commentaires P1.
5. Versionner les livraisons avec SemVer et des tags signés : `v0.1.0`, `v0.2.0`, `v1.0.0`. Produire un `CHANGELOG.md` depuis les commits conventionnels. La version frontend doit être mise à jour lors d’une release; ajouter une version API ou une image Docker taguée par version et SHA Git.

## Écarts de production prioritaires

### P0 — avant tout déploiement

- Créer des réglages séparés `config/settings/development.py`, `production.py` et `test.py`. En production, `DEBUG=False`, `SECRET_KEY` obligatoire sans valeur par défaut, `ALLOWED_HOSTS`, CORS et origines CSRF explicitement définis, HTTPS/HSTS, cookies sécurisés et journalisation structurée.
- Ne pas utiliser `runserver`, les volumes de code ni les ports PostgreSQL/Redis publiés dans la composition de production. Construire des images immuables et multi-étapes ; lancer Django via Gunicorn derrière Nginx, avec `collectstatic` pendant le déploiement.
- Ajouter des services `celery-worker` (et `celery-beat` seulement si des tâches planifiées existent), des health checks applicatifs, des redémarrages contrôlés, volumes persistants et une stratégie de sauvegarde/restauration PostgreSQL testée.
- Corriger les risques hors-ligne détaillés dans `REVUE-CORRECTIONS-HORS-LIGNE.md`, en particulier le cache API partagé, l’idempotence et la réconciliation des identifiants temporaires.

### P1 — fiabiliser la livraison

- Étendre la CI : `npm run build` manque actuellement ; ajouter contrôle des migrations (`makemigrations --check`), exécution des migrations, tests backend/ frontend, Ruff format/check, audit des dépendances et scan de secrets. Épingler les actions GitHub par SHA ou politique organisationnelle.
- Ajouter une image de production et une étape de publication dans un registre, avec tags `version` et `sha-<commit>`. Déployer uniquement l’image validée par la CI.
- Centraliser les secrets dans le gestionnaire de secrets de l’environnement de déploiement ; `.env.example` ne contient que des valeurs factices. Ne jamais committer `.env`, tokens, exports de base ou clés privées.
- Mettre en place observabilité et exploitation : logs JSON avec identifiant de requête/audit, suivi des erreurs, métriques (latence, taux d’échec, files Celery), alertes, runbook d’incident et procédure de rollback.

### P2 — qualité durable

- Définir des seuils de couverture et ajouter des tests d’intégration pour les transitions de panne, permissions, audit et synchronisation. Les RB-TEST-001 et RB-CL-001 doivent être testées côté serveur.
- Ajouter des contrôles d’accessibilité, de performance et de dépendances; tester le PWA sur appareil partagé et hors-ligne.
- Documenter les politiques de rétention, sauvegarde, comptes de service, moindre privilège et rotation des secrets.

## Manière de travailler professionnelle

1. Partir d’une issue courte : objectif, règles métier concernées, critères d’acceptation, risques et plan de test.
2. Créer une branche dédiée et synchroniser `main` avant de coder. Ne pas mélanger refactorisation, fonctionnalité et changement de dépendances dans une même PR sans nécessité.
3. Implémenter par couches : domaine et tests, application/API, interface, puis documentation et migration. Toute transition d’état est validée par le serveur et laisse une trace d’audit.
4. Avant revue, exécuter depuis les bons répertoires : `pytest` dans `backend/`; `npm run lint`, `npm test` et `npm run build` dans `frontend/`; puis `git status` pour repérer les fichiers oubliés.
5. Ouvrir une PR ciblée : expliquer le besoin, les changements, migrations, tests exécutés, captures pour l’UI et plan de retour arrière. Lier l’issue et demander une revue.
6. Fusionner seulement lorsque la CI est verte, les migrations sont sûres, les commentaires résolus et le déploiement/rollback préparés. Taguer la release, publier les notes de version et surveiller l’application après livraison.

## Définition de « prêt pour production »

Une version est prête lorsque le code, tests, migrations et documentation sont tous suivis par Git; la CI construit l’interface et teste l’API sur PostgreSQL; les secrets et paramètres production sont externes; l’application s’exécute en images immuables derrière HTTPS; sauvegardes, alertes et rollback ont été testés; et les règles métier ainsi que l’isolation des données hors-ligne sont vérifiées.
