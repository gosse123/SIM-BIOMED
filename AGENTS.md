# AGENTS.md — SIM-BIOMED

## What this repo is

Documentation-only repo for SIM-BIOMED (Système Intelligent de Maintenance Biomédicale). No application code exists yet — the repo contains architecture specs, business rules, UML diagrams, and UI mockups.

## Key files

- `doc/SIM-BIOMED-Architecture-Technique-MVP.md` — Technical architecture and MVP scope
- `doc/SIM-BIOMED-Regles-Metier-Strictes.md` — Normative business rules (RB-* codes)
- `doc/SIM-BIOMED — Architecture logicielle(1).md` — Layered architecture design
- `doc/SIM-BIOMED — Logique métier(1).md` — Business logic and domain model
- `doc/00-classes.puml` — UML class diagram (PlantUML)
- `doc/disigne-market/` — UI mockups (HTML screenshots)

## Target stack (from architecture doc)

- **Frontend:** React + TypeScript + Tailwind CSS + Vite
- **Backend:** Django + Django REST Framework + PostgreSQL
- **Async:** Celery + Redis
- **Infra:** Docker Compose, Nginx, JWT auth
- **Tests:** pytest/pytest-django (backend), Vitest/Testing Library (frontend)
- **CI:** GitHub Actions

## Architecture rules (non-negotiable)

- **Monolithic modular** architecture — no microservices in MVP
- **Layered:** API → Application → Domain → Infrastructure
- Domain layer must not depend on React, HTTP, or Django views
- Server is the authority — no business rules enforced only in frontend
- All state transitions must be explicit domain-controlled (`backend/domain/`)
- Every important action leaves an audit trace
- No closure without test result (RB-TEST-001, RB-CL-001)
- **Cloisonnement multi-établissements** : tout queryset est filtré par `etablissement` de l'utilisateur via `apps/accounts/scoping.py::scope_to_etablissement`. Un utilisateur sans établissement (super-admin d'onboarding) voit tout. L'établissement d'un utilisateur est attribué par l'admin à l'approbation — jamais choisi librement au profil
- Le mot de passe temporaire n'est jamais persisté (pas dans Notification) — retourné uniquement dans la réponse API d'approbation

## MVP scope constraints

- No ML/predictive models (RB-SCOPE-004)
- No microservices (RB-SCOPE-003)
- No native mobile app — responsive web/PWA covers field needs (RB-SCOPE-007)
- No patient record features (RB-SCOPE-005)
- No Kubernetes (RB-SCOPE-008)

## Business domain vocabulary

Equipment statuses: `FONCTIONNEL`, `FONCTIONNEL_SOUS_SURVEILLANCE`, `EN_PANNE`, `EN_MAINTENANCE`, `EN_ATTENTE_PIECE_OU_PRESTATAIRE`, `HORS_SERVICE`, `REFORME`

Failure statuses: `SIGNALEE`, `QUALIFIEE`, `CRITICITE_EVALUEE`, `EN_DIAGNOSTIC`, `EN_INTERVENTION`, `EN_TEST`, `EN_ATTENTE_PIECE`, `EN_ATTENTE_PRESTATAIRE`, `CLOSE`

Test results: `CONFORME`, `SOUS_SURVEILLANCE`, `NON_CONFORME`, `TOUJOURS_EN_PANNE`

## Language

All documentation and domain terminology is in French. Use French domain terms when working on this codebase.
