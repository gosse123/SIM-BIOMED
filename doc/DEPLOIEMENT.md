# Déploiement SIM-BIOMED (MVP)

Cible retenue : **PaaS managé (Render / Railway / Fly)**, **PostgreSQL externe
(Supabase ou équivalent)**, publication d'images par la plateforme (pas de
registre d'images géré en CI pour le MVP).

## 1. Vue d'ensemble

```text
Navigateur ──HTTPS──▶ Frontend (nginx, Dockerfile.prod)
                          │  /api, /admin, /static
                          ▼
                     API (Django + Gunicorn, Dockerfile.prod) ──▶ PostgreSQL externe
                          │                                        (Supabase)
                          ├──▶ Worker Celery (mêmes sources)
                          └──▶ Beat Celery (planification préventive)
                                        ▲
                                        └── Render Key Value (broker, fourni par le blueprint)
```

- **TLS** : terminé par la plateforme. Django reçoit `X-Forwarded-Proto` et ne
  doit jamais être exposé directement.
- **Frontend** : conteneur nginx qui sert le build Vite et proxyse `/api`,
  `/admin`, `/static` vers l'API via `API_UPSTREAM`. Le code applicatif utilise
  une `baseURL` relative `/api`, donc aucun rebuild n'est nécessaire pour
  changer d'environnement.
- **Trois services backend** partagent la même image : `api` (web), `worker`
  (Celery), `beat` (planning). Seul l'`api` exécute migrations et
  collectstatic au démarrage.

## 2. Artefacts de déploiement

| Fichier | Dépôt | Rôle |
| --- | --- | --- |
| `Dockerfile.prod` | backend | Image multi-étapes, utilisateur non-root, `ENTRYPOINT` migrations + `collectstatic`, `CMD` Gunicorn sur `$PORT` |
| `docker-entrypoint.sh` | backend | Migrations avec 5 tentatives (attente base), `collectstatic`, puis `CMD` ; désactivables par `RUN_MIGRATIONS=false` / `RUN_COLLECTSTATIC=false` |
| `render.yaml` | backend | Blueprint : `api`, `worker`, `beat` **+ Key Value (broker)**, tous en `region: frankfurt` |
| `render.free.yaml` | backend | Variante **test gratuit** : `api` seul, sans worker/beat ni Key Value |
| `.env.example` | backend | Catalogue des variables (local et production), valeurs factices |
| `GET /api/healthz/` et `/healthz/` | backend | Sonde : `200` base OK, `503` base inaccessible, accessible sans authentification |
| `Dockerfile.prod` | frontend | Build Vite puis `nginx:alpine` |
| `nginx/default.conf.template` | frontend | SPA + proxy `/api` (`API_UPSTREAM`), cache assets, `/healthz` |
| `render.yaml` | frontend | Blueprint du service web |

Les fichiers `Dockerfile` et `docker-compose.yml` restent réservés au
**développement local** (`runserver`, ports exposés, volumes de code).

## 3. Variables d'environnement

### Backend (`config.settings.production`)

| Variable | Obligatoire | Description |
| --- | --- | --- |
| `DJANGO_SECRET_KEY` | oui | Jamais par défaut en production (levée si absente) |
| `ALLOWED_HOSTS` | oui | Hôtes de l'API, séparés par des virgules. `RENDER_EXTERNAL_HOSTNAME` (injecté par Render) est **ajouté automatiquement**, donc un suffixe de sous-domaine ne casse pas le health check |
| `FRONTEND_ORIGIN` | oui | Origine de l'interface, ex. `https://simbiomed-frontend.onrender.com` — alimente à la fois `CORS_ALLOWED_ORIGINS` et `CSRF_TRUSTED_ORIGINS` (`config/settings/origins.py`). Sans elle : **403 CSRF** sur toute requête POST du navigateur, car le proxy nginx envoie `Host: api` |
| `CSRF_TRUSTED_ORIGINS` | oui | Origine **propre** de l'API (repli hors Render) ; l'originale du front vient de `FRONTEND_ORIGIN` |
| `DATABASE_URL` | oui* | `postgresql://USER:PASSWORD@HOST:5432/postgres?sslmode=require` — **pooler Supabase en mode SESSION, jamais Transaction** (le mode Transaction casse les prepared statements de Django) |
| `REDIS_URL` | oui | Broker Celery ; renseigné **automatiquement** par le blueprint via `fromService` (Key Value) |
| `CORS_ALLOWED_ORIGINS` | non | Origines CORS supplémentaires, en plus de `FRONTEND_ORIGIN` |
| `DJANGO_SETTINGS_MODULE` | oui | `config.settings.production` |
| `SECURE_SSL_REDIRECT` | non (défaut `true`) | `false` uniquement si le proxy ne transmet pas `X-Forwarded-Proto` |
| `LOG_LEVEL` | non (défaut `WARNING`) | `INFO` recommandé en production |
| `PORT` | non (défaut `8000`) | Fourni par la plateforme |
| `WEB_CONCURRENCY` | non (défaut `2`) | Workers Gunicorn |
| `RUN_MIGRATIONS` / `RUN_COLLECTSTATIC` | non (défaut `true`) | `false` sur worker/beat |

\* Alternative à `DATABASE_URL` : `DB_HOST`, `DB_PORT`, `POSTGRES_DB`,
`POSTGRES_USER`, `POSTGRES_PASSWORD` (mutuellement exclusifs).

### Frontend

| Variable | Obligatoire | Description |
| --- | --- | --- |
| `API_UPSTREAM` | oui | URL de l'API, par ex. `https://simbiomed-api.onrender.com` ou `http://simbiomed-api:10000` en réseau privé |

## 4. Mise en route (Render)

1. **Base de données (Supabase)**
   - Créer le projet, copier l'URL de connexion en mode **Pooler SESSION**
     (jamais *Transaction* : Django utilise des prepared statements), ajouter
     `?sslmode=require` → future `DATABASE_URL`.
   - Activer les sauvegardes automatiques et noter où se fait la restauration.
2. **Région** : les trois services et le Key Value sont déclarés en
   `region: frankfurt`, la zone la plus proche de Supabase (`eu-west-2`,
   Londres). **La région est figée après la première création** — ne pas la
   modifier au prix d'une latence de ~150 ms par aller-retour.
3. **API** : *New Blueprint* sur le dépôt backend, garder `render.yaml`.
   - Renseigner **`DATABASE_URL` seule** (marquée `sync: false`) — jamais dans
     le dépôt. `REDIS_URL`, `DJANGO_SECRET_KEY` sont produits par le blueprint.
   - `ALLOWED_HOSTS` et l'origine propre de l'API sont complétés
     automatiquement via `RENDER_EXTERNAL_HOSTNAME`. Seule variable à ajuster
     manuellement si Render suffixe le sous-domaine du front :
     `FRONTEND_ORIGIN`.
   - Le premier démarrage exécute `migrate` puis `collectstatic`. Les
     tout premiers health checks peuvent renvoyer `503` pendant le
     réchauffement du pool de connexions : c'est transitoire (≈ 1 min).
4. **Worker et Beat** : déclarés par le même blueprint ; ils récupèrent
   `DJANGO_SECRET_KEY` et `DATABASE_URL` via `fromService` sur l'API, et
   `REDIS_URL` sur l'instance **Key Value** `simbiomed-redis`
   (`ipAllowList: []` = réseau privé, `maxmemoryPolicy: noeviction`).
5. **Frontend** : *New Blueprint* sur le dépôt frontend, garder `render.yaml`,
   renseigner `API_UPSTREAM` avec l'URL de l'API (même région → 2 ms).
6. **Compte initial** : console du service API →
   `python manage.py createsuperuser`, puis approuver les demandes d'accès
   depuis l'interface (cloisonnement par établissement, RB-*).

### Vérification post-déploiement

```bash
curl -fsS https://API_HOST/api/healthz/      # {"status": "ok", "database": "up"}
curl -fsS https://FRONTEND_HOST/healthz      # ok
curl -fsSI https://FRONTEND_HOST/            # 200, index.html
# Connexion réelle : POST /api/auth/login/ depuis l'interface
```

## 5. Test gratuit (Render free + Supabase)

Pour un essai sans budget : deux services web gratuits + base Supabase.

| Élément | Plateforme | Tarif |
| --- | --- | --- |
| `api` + `frontend` | Render, tier gratuit (750 h/mois cumulées) | 0 |
| PostgreSQL | Supabase free (500 Mo) | 0 |
| `worker` / `beat` | omit (background workers payants) | — |
| Redis | inutile : rien ne parle à Celery sans worker | — |

### Procédure

1. **Supabase** → *New project* → *Settings → Database → Connection string (URI)* :
   copier l'URL, mode **Pooler SESSION** (pas Transaction), et ajouter
   `?sslmode=require` si absent.
2. **Backend** → *New Blueprint* sur `SIM-BIOMED-backend` → sélectionner le
   fichier blueprint **`render.free.yaml`**
   (si le tableau de bord ne propose pas de choix de fichier, il lit
   `render.yaml` : basculer temporairement avec
   `git mv render.yaml render.full.yaml && git mv render.free.yaml render.yaml`,
   puis inverser au moment de passer au complet).
3. Renseigner `DATABASE_URL` (marqué `sync: false`) — à saisir dans le
   dashboard Render, **jamais** dans le dépôt.
4. **Frontend** → *New Blueprint* sur `SIM-BIOMED-frontend`, garder
   `render.yaml`, vérifier `API_UPSTREAM = https://SIMBIOMED-API.onrender.com`.
5. Après le premier déploiement : console du service `api` →
   `python manage.py createsuperuser`.
6. Vérifications : `curl -fsS https://API/api/healthz/`, puis la connexion réelle
   depuis l'interface (un **403 « Vérification CSRF a échoué »** au login signale
   que `FRONTEND_ORIGIN` pointe vers le mauvais domaine).

### Tâches Celery à la main (pas de beat en gratuit)

```bash
python manage.py shell -c "from apps.preventive.tasks import check_overdue_maintenance as f; print(f())"
python manage.py shell -c "from apps.preventive.tasks import generate_preventive_schedule as f; print(f())"
```

### Limites du tier gratuit

- Veille après ~15 min sans activité → premier appel à 30-60 s (attendre le
  premier appel avant de conclure à une panne).
- 750 h/mois cumulées sur les services gratuits : suffisant pour un test en
  continu d'un ou deux services.
- Domaine `*.onrender.com` avec HTTPS ; domaine personnalisé = payant.
- `WEB_CONCURRENCY=1` (512 Mo de RAM) — déjà posé dans `render.free.yaml`.

## 6. Commandes utiles

```bash
# Images de production en local
docker build -f Dockerfile.prod -t simbiomed-api .          # backend
docker build -f Dockerfile.prod -t simbiomed-frontend .     # frontend

# Console / maintenance (Web Shell de la plateforme)
python manage.py migrate
python manage.py createsuperuser
python manage.py collectstatic --noinput
python manage.py shell

# Vérifications avant livraison (équivalent CI)
ruff check . && pytest tests/ -v          # backend
npm run lint && npm run test && npm run build   # frontend
```

## 7. Variantes Railway / Fly

- **Railway** : créer deux services à partir des dépôts, builder avec
  `Dockerfile.prod`, commande web `sh -c "gunicorn …"` (déjà le `CMD` de
  l'image), worker `celery -A config worker -l info`, beat
  `celery -A config beat -l info`. La plateforme fournit `PORT` et
  `DATABASE_URL`.
- **Fly.io** : un `fly init` par service, `internal_port` aligné sur `$PORT`,
  health check sur `/api/healthz/` (api) et `/healthz` (frontend).
- Dans les deux cas : PostgreSQL externe obligatoire (les services n'exposent
  aucun port de base).

## 8. Chaîne de proxy et HTTPS

```text
Navigateur ──https──▶ Proxy plateforme ──http + X-Forwarded-Proto: https──▶ nginx ──▶ Gunicorn
```

- Django lit `X-Forwarded-Proto` (`SECURE_PROXY_SSL_HEADER`) : sans cet
  en-tête, `SECURE_SSL_REDIRECT` renvoie une redirection HTTPS qui, remontée à
  travers nginx, provoquerait une **boucle**. Le template nginx propage
  l'en-tête reçu (`map $http_x_forwarded_proto`) — ne pas le supprimer.
- Si le proxy ne fournit jamais cet en-tête : `SECURE_SSL_REDIRECT=false`.
- Le `HEALTHCHECK` du conteneur envoie explicitement `X-Forwarded-Proto:
  https` pour ne pas être victime de la redirection.

## 9. Secrets et configuration

- Les secrets vivent uniquement dans le gestionnaire de secrets de la
  plateforme ; `.env.example` ne contient que des valeurs factices.
- Ne jamais commiter `.env`, ni de dump de base (`db.sqlite3` est encore suivi
  dans le dépôt backend : à retirer du suivi, voir
  `ANALYSE-STRUCTURE-VERSION-PRODUCTION.md` §Structure).
- Rotation : régénérer `DJANGO_SECRET_KEY` invalide les JWT en cours — à
  prévoir en heure creuse.

## 10. Sauvegardes et restauration

- **Supabase** : sauvegardes automatiques (PITR) à activer ; test de
  restauration sur un instantané au moins une fois avant la mise en service.
- Export manuel :

```bash
pg_dump "$DATABASE_URL" | gzip > simbiomed-$(date +%F).sql.gz
# Restauration de contrôle sur une base vierge
gunzip -c simbiomed-YYYY-MM-DD.sql.gz | psql "$DATABASE_URL"
```

## 11. Déploiement continu, supervision, rollback

- La CI (GitHub Actions) valide chaque push/PR : `ruff`, `pytest` +
  contrôle des migrations (backend), `lint` + `tests` + `build` (frontend).
- `autoDeployTrigger: 'commit'` déploie la branche `main` : ne merger que CI
  verte et
  migration sûre (migrations rétro-compatibles : colonnes ajoutées puis
  suppressées au déploiement suivant).
- **Supervision** : logs JSON côté Django (`LOG_LEVEL`), health checks de la
  plateforme sur `/api/healthz/` et `/healthz`, à surveiller : taux 5xx,
  erreurs Celery, latence.
- **Rollback** : redeployer le commit/segment précédent depuis l'historique
  de déploiements de la plateforme, puis `python manage.py migrate` si un
  retour en arrière de schéma est nécessaire (éviter les migrations destructives).

## 12. Checklist « prêt pour production »

Liée à `ANALYSE-STRUCTURE-VERSION-PRODUCTION.md` :

- [x] Réglages `development` / `production` / `test` distincts, secrets requis
- [x] Images immuables multi-étapes, Gunicorn, pas de `runserver` ni de port
      PostgreSQL/Redis exposé
- [x] Services `worker` et `beat` déclarés, health checks applicatifs
- [x] Fichiers statiques servis (`collectstatic` + WhiteNoise)
- [x] CI : tests, build frontend, contrôle des migrations
- [ ] Sauvegardes **restaurées** au moins une fois (§10)
- [ ] `db.sqlite3` retiré du suivi Git
- [ ] Correctifs `REVUE-CORRECTIONS-HORS-LIGNE.md` validés avant ouverture
      aux utilisateurs
