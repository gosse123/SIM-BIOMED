# Corrections requises après revue

## Portée

Cette note décrit les sept problèmes bloquants relevés sur la branche et la correction attendue. Les trois premiers sont des omissions du diff : les fichiers existent localement mais sont non suivis (`git status` les affiche avec `??`). Ils doivent être ajoutés au même commit que les fichiers qui les importent ou les utilisent.

## 1. Module d'audit absent du diff

**Emplacement :** `backend/config/settings/base.py` et imports de `backend/apps/accounts/views.py`.

**Problème :** Django charge `apps.audit.models`, mais le diff ne livre pas l'implémentation. L'application échoue au chargement des URL.

**Correction :** ajouter au commit `backend/apps/audit/models.py`, `admin.py`, `migrations/0001_initial.py` et les fichiers `__init__.py`. Vérifier que `apps.audit` est dans `INSTALLED_APPS`, puis exécuter `python manage.py migrate` et `pytest` depuis `backend/`.

## 2. Migrations de schéma absentes

**Emplacement :** `backend/apps/accounts/models.py` et `backend/apps/equipment/models.py`.

**Problème :** les nouveaux modèles et champs (`Etablissement`, demandes d'accès, notifications, clés étrangères) n'existent pas dans une base déjà migrée depuis `main`.

**Correction :** ajouter les migrations générées : `accounts/migrations/0002_*` à `0004_*` et `equipment/migrations/0002_*`. Relire les dépendances et les opérations de données avant livraison, puis valider avec `python manage.py makemigrations --check`, `python manage.py migrate` et les tests backend sur une base vide et une base migrée.

## 3. Pages React absentes du diff

**Emplacement :** imports de `frontend/src/app/App.tsx`.

**Problème :** les routes importent `EquipmentFormPage`, `ProfilePage`, `UsersPage`, `DemandesPage` et `CompleteProfilePage`, qui ne figurent pas dans le changement publié. Le build TypeScript échoue.

**Correction :** inclure ces cinq fichiers, ainsi que leurs composants, types et utilitaires non suivis nécessaires. Exécuter `npm run build`, `npm run lint` et `npm test` dans `frontend/` avant le commit.

## 4. Clé d'idempotence remplacée pendant la synchronisation

**Emplacement :** intercepteur de requête dans `frontend/src/services/api.ts`.

**Problème :** `processSyncQueue()` fournit `X-Offline-Id: entry.id`, puis l'intercepteur le remplace par un UUID neuf. Une reprise après réponse perdue peut donc créer une panne en double.

**Correction :** ne générer une clé que si l'en-tête est absent : `config.headers['X-Offline-Id'] ??= uuidv4()`. Préserver également l'en-tête lors de la reprise après un 401. Ajouter un test qui rejoue la même entrée et vérifie une seule mutation côté serveur.

## 5. Cache de listes lu avec le mauvais type

**Emplacement :** `frontend/src/services/equipment.ts`, `panne.ts` et `intervention.ts`.

**Problème :** une liste est enregistrée comme tableau sous `cacheKey`, mais le repli appelle `getAll<T>()`. Il retourne donc des tableaux imbriqués et mélange listes filtrées et fiches détail.

**Correction :** importer `get` et lire la clé exacte : `const cached = await get<Equipment[]>('equipment', cacheKey); return cached ?? []`. Appliquer le même schéma aux pannes et interventions. Pour les détails, utiliser `get<T>(store, detailKey)`. Tester réseau indisponible, erreur réseau et paramètres de filtre distincts.

## 6. Réponses API authentifiées partagées entre utilisateurs

**Emplacement :** `frontend/public/sw.js`, fonction `networkFirst()` pour `/api/`.

**Problème :** le cache du service worker est commun à l'appareil et indexé par URL ; en hors-ligne, un second utilisateur peut recevoir `/api/auth/me/` ou des données biomédicales du précédent.

**Correction :** ne jamais mettre les réponses `/api/` dans Cache Storage : pour ces requêtes, retourner la réponse réseau ou une erreur 503 hors-ligne. Conserver le cache hors-ligne dans IndexedDB, avec une base ou des clés préfixées par l'identifiant de l'utilisateur ; vider ce stockage et la file de synchronisation à la déconnexion. Ajouter un test de déconnexion/changement d'utilisateur.

## 7. Entités temporaires hors-ligne non persistées ni réconciliées

**Emplacement :** `offlineAwareRequest()` dans `frontend/src/services/api.ts` et `ReportPannePage.tsx`.

**Problème :** le rapport de panne renvoie un identifiant négatif et la page navigue vers son détail, mais cette entité n'est pas mise en cache. Après synchronisation, aucun lien temporaire → identifiant serveur n'est conservé ; les opérations dépendantes ciblent encore une URL négative.

**Correction :** à la création hors-ligne, enregistrer l'entité temporaire dans `pending-entities` et dans le cache métier sous `panne-${tempId}`. Ajouter `temporaryId` à l'entrée de file. Lors du succès de synchronisation, lire la réponse serveur, remplacer le cache temporaire par l'entité serveur, stocker le mapping temporaire → réel, puis réécrire les URL et corps des entrées dépendantes avant leur envoi. La page détail doit afficher l'état « en attente de synchronisation » et rediriger vers l'identifiant réel après réconciliation. Tester création hors-ligne, consultation immédiate, reprise réseau et action dépendante.

## Critère de livraison

Ne fusionner qu'après ajout de tous les fichiers non suivis requis, migrations appliquées avec succès, et passage des commandes `pytest`, `npm run build`, `npm run lint` et `npm test`.
