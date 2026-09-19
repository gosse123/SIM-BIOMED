# SIM-BIOMED --- Architecture technique MVP

## 1. Objet du document

Ce document transforme la logique métier et l'architecture logicielle de
SIM-BIOMED en une architecture technique concrète pour le MVP.

Le MVP doit rester volontairement simple, modulaire, testable et
évolutif. Il ne doit pas anticiper prématurément les besoins des niveaux
avancés.

## 2. Vision du MVP

SIM-BIOMED est un système de gestion de maintenance biomédicale
permettant de :

1.  gérer le parc d'équipements ;
2.  enregistrer et suivre les pannes ;
3.  organiser les interventions ;
4.  gérer la maintenance préventive ;
5.  calculer les principaux indicateurs ;
6.  gérer la criticité et la priorisation ;
7.  conserver un historique et une traçabilité des actions.

La logique métier reste au centre du système.

## 3. Décision architecturale

### Architecture retenue

**Monolithe modulaire avec architecture en couches.**

Le MVP ne sera pas développé en microservices.

### Raisons

-   le domaine est encore en phase de validation ;
-   les modules partagent beaucoup de données et de règles ;
-   une base transactionnelle unique simplifie la cohérence ;
-   le déploiement est plus simple ;
-   les tests sont plus simples ;
-   le coût d'exploitation est plus faible ;
-   une séparation ultérieure reste possible si un besoin réel apparaît.

### Règle

> Aucune extraction en microservice ne doit être faite uniquement pour
> des raisons de mode ou de sophistication technique.

## 4. Stack technique

  -----------------------------------------------------------------------
  Domaine                 Technologie             Rôle
  ----------------------- ----------------------- -----------------------
  Frontend                React                   Interface web

  Langage frontend        TypeScript              Typage statique

  UI                      Tailwind CSS            Système de styles

  Build                   Vite                    Développement et build
                                                  frontend

  Backend                 Django                  Application serveur

  API                     Django REST Framework   API REST

  Base de données         PostgreSQL              Persistance principale

  Authentification API    JWT                     Authentification

  Tâches asynchrones      Celery                  Jobs et traitements
                                                  différés

  Broker/cache            Redis                   File de tâches et cache

  Reverse proxy           Nginx                   HTTPS, routage et
                                                  exposition

  Conteneurisation        Docker / Docker Compose Environnement
                                                  reproductible

  Tests backend           pytest / pytest-django  Tests

  Tests frontend          Vitest / Testing        Tests UI
                          Library                 

  CI                      GitHub Actions          Vérification
                                                  automatique
  -----------------------------------------------------------------------

## 5. Architecture logique

``` text
Utilisateur
    |
    v
React + TypeScript
    |
    | HTTPS / REST
    v
Nginx
    |
    v
Django REST Framework
    |
    v
Application Services
    |
    v
Domain métier
    |
    v
Infrastructure / ORM
    |
    v
PostgreSQL
```

Les traitements différés suivent :

``` text
Django
   |
   v
Celery
   |
   v
Redis
```

## 6. Découpage backend

``` text
backend/
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   ├── asgi.py
│   └── celery.py
│
├── apps/
│   ├── accounts/
│   ├── equipment/
│   ├── failures/
│   ├── interventions/
│   ├── preventive/
│   ├── indicators/
│   ├── alerts/
│   └── audit/
│
├── domain/
│   ├── equipment/
│   ├── failure/
│   ├── maintenance/
│   ├── criticality/
│   ├── prioritization/
│   └── indicators/
│
├── infrastructure/
│   ├── storage/
│   ├── notifications/
│   └── authentication/
│
├── tests/
└── manage.py
```

## 7. Responsabilités des couches

### API

Responsable de :

-   recevoir les requêtes ;
-   authentifier ;
-   autoriser ;
-   valider les entrées de transport ;
-   appeler les services applicatifs ;
-   retourner les réponses HTTP.

L'API ne doit pas contenir les règles métier complexes.

### Application

Responsable de l'orchestration des cas d'utilisation.

Exemples :

-   créer un équipement ;
-   signaler une panne ;
-   qualifier une panne ;
-   démarrer une intervention ;
-   enregistrer un test ;
-   clôturer une panne.

### Domaine

Responsable des règles métier.

Exemples :

-   transitions de statut ;
-   calcul de criticité ;
-   calcul de priorité ;
-   conditions de clôture ;
-   règles de remise en service.

Le domaine ne doit pas dépendre de React, HTTP ou d'une vue Django.

### Infrastructure

Responsable des détails techniques :

-   PostgreSQL ;
-   stockage de fichiers ;
-   e-mail ;
-   Redis ;
-   authentification externe éventuelle.

## 8. Modèle de données MVP

### Entités principales

``` text
User
Role
Service
Location

Equipment
EquipmentEvent

Failure
Diagnosis
Intervention
InterventionPart
SparePart

PreventiveMaintenance

Indicator
Alert
AuditLog
```

### Relations principales

``` text
Service
   |
   +---- Equipment
   |
   +---- User

Location
   |
   +---- Equipment

Equipment
   |
   +---- EquipmentEvent
   +---- Failure
   +---- PreventiveMaintenance

Failure
   |
   +---- Diagnosis
   +---- Intervention

Intervention
   |
   +---- InterventionPart ---- SparePart
```

## 9. API MVP

Ressources initiales :

``` text
/api/auth/
/api/equipment/
/api/services/
/api/locations/
/api/failures/
/api/interventions/
/api/preventive-maintenance/
/api/indicators/
/api/alerts/
/api/audit/
```

Exemples :

``` text
GET    /api/equipment/
POST   /api/equipment/
GET    /api/equipment/{id}/
PATCH  /api/equipment/{id}/

POST   /api/failures/
GET    /api/failures/
GET    /api/failures/{id}/

POST   /api/failures/{id}/qualify/
POST   /api/failures/{id}/diagnose/
POST   /api/failures/{id}/interventions/
POST   /api/failures/{id}/test/
POST   /api/failures/{id}/close/
```

Les actions métier importantes doivent être représentées comme des
opérations métier explicites lorsque cela améliore la lisibilité et la
sécurité.

## 10. États métier

### Équipement

``` text
FONCTIONNEL
FONCTIONNEL_SOUS_SURVEILLANCE
EN_PANNE
EN_MAINTENANCE
EN_ATTENTE
HORS_SERVICE
REFORME
```

### Panne

``` text
SIGNALEE
QUALIFIEE
CRITICITE_EVALUEE
EN_DIAGNOSTIC
EN_INTERVENTION
EN_TEST
EN_ATTENTE_PIECE
EN_ATTENTE_PRESTATAIRE
CLOSE
```

Toutes les transitions doivent être contrôlées par le domaine.

## 11. Authentification et autorisation

Rôles MVP :

-   Administrateur ;
-   Responsable biomédical ;
-   Technicien ;
-   Personnel soignant ;
-   Direction.

Le contrôle d'accès doit être effectué côté serveur.

Le frontend ne constitue jamais une frontière de sécurité.

## 12. Historique et audit

Deux concepts doivent rester distincts :

### Historique métier

Conserve les événements liés à l'équipement et à sa maintenance.

### Audit

Conserve les actions sensibles réalisées par les utilisateurs.

Exemples d'actions auditées :

-   changement de statut ;
-   modification de criticité ;
-   clôture ;
-   réforme ;
-   modification importante d'un équipement.

## 13. Traitements planifiés

Celery sera utilisé pour les traitements qui n'ont pas besoin de bloquer
une requête utilisateur.

Exemples :

-   mise à jour des échéances préventives ;
-   calcul périodique des indicateurs ;
-   génération d'alertes ;
-   analyse de tendances lorsqu'elle sera introduite.

Le MVP ne doit pas introduire de machine learning.

## 14. Sécurité minimale

-   HTTPS en production ;
-   mots de passe gérés par Django ;
-   JWT avec expiration ;
-   RBAC ;
-   validation serveur ;
-   protection des endpoints ;
-   journalisation des actions sensibles ;
-   variables secrètes hors du dépôt ;
-   sauvegardes PostgreSQL ;
-   limitation raisonnable des requêtes sensibles ;
-   contrôle des fichiers envoyés.

## 15. Frontend

Structure indicative :

``` text
frontend/
├── src/
│   ├── app/
│   ├── components/
│   ├── features/
│   │   ├── equipment/
│   │   ├── failures/
│   │   ├── interventions/
│   │   ├── preventive/
│   │   ├── indicators/
│   │   └── alerts/
│   ├── layouts/
│   ├── pages/
│   ├── services/
│   ├── types/
│   └── utils/
└── ...
```

Le frontend doit être organisé par fonctionnalités plutôt que par une
collection globale de composants sans responsabilité claire.

## 16. Pages MVP

### Authentification

-   Connexion.

### Dashboard

-   nombre d'équipements ;
-   pannes ;
-   maintenances ;
-   disponibilité ;
-   MTBF ;
-   MTTR ;
-   alertes importantes.

### Parc

-   liste ;
-   création ;
-   modification ;
-   détail ;
-   historique.

### Pannes

-   signalement ;
-   liste ;
-   détail ;
-   qualification ;
-   diagnostic ;
-   intervention ;
-   test ;
-   clôture.

### Maintenance préventive

-   liste ;
-   échéances ;
-   retard ;
-   détail.

### Indicateurs

-   MTBF ;
-   MTTR ;
-   disponibilité ;
-   taux de panne.

## 17. Plan de développement

### Phase 0 --- Fondation

-   dépôt Git ;
-   structure backend/frontend ;
-   Docker Compose ;
-   PostgreSQL ;
-   Django ;
-   React ;
-   CI ;
-   variables d'environnement ;
-   conventions de code.

**Critère de sortie :**

Frontend → API → PostgreSQL fonctionnent ensemble.

### Phase 1 --- Utilisateurs

-   modèle utilisateur ;
-   rôles ;
-   permissions ;
-   connexion ;
-   JWT.

**Critère de sortie :**

Un utilisateur ne peut accéder qu'aux fonctionnalités autorisées par son
rôle.

### Phase 2 --- Parc

-   Service ;
-   Location ;
-   Equipment ;
-   EquipmentEvent ;
-   CRUD ;
-   recherche ;
-   filtres ;
-   historique.

**Critère de sortie :**

Un équipement peut être créé, consulté, modifié et suivi.

### Phase 3 --- Pannes

-   Failure ;
-   qualification ;
-   criticité ;
-   états ;
-   transitions ;
-   historique.

**Critère de sortie :**

Une panne suit uniquement les transitions autorisées.

### Phase 4 --- Interventions

-   Diagnosis ;
-   Intervention ;
-   SparePart ;
-   InterventionPart ;
-   tests ;
-   clôture.

**Critère de sortie :**

Une panne ne peut pas être clôturée sans résultat de test.

### Phase 5 --- Priorisation

-   moteur de criticité ;
-   moteur de priorité ;
-   file de travail.

**Critère de sortie :**

Les interventions sont ordonnées selon les règles métier définies.

### Phase 6 --- Maintenance préventive

-   périodicités ;
-   échéances ;
-   états ;
-   historique ;
-   jobs Celery.

**Critère de sortie :**

Les échéances sont calculées et les retards détectés.

### Phase 7 --- Indicateurs

-   MTBF ;
-   MTTR ;
-   disponibilité ;
-   indisponibilité ;
-   taux de panne ;
-   agrégations.

**Critère de sortie :**

Les indicateurs sont calculés à partir des données métier et testés.

### Phase 8 --- Dashboard et alertes

-   dashboard ;
-   graphiques ;
-   alertes ;
-   notifications.

### Phase 9 --- Qualité

-   tests ;
-   sécurité ;
-   audit ;
-   sauvegardes ;
-   documentation ;
-   tests d'intégration.

### Phase 10 --- Production

-   Docker production ;
-   Nginx ;
-   HTTPS ;
-   PostgreSQL ;
-   monitoring ;
-   sauvegardes ;
-   procédure de restauration.

## 18. Définition de "MVP terminé"

Le MVP est terminé uniquement lorsque :

-   les équipements peuvent être gérés ;
-   les pannes peuvent être signalées ;
-   les pannes peuvent être qualifiées ;
-   la criticité peut être évaluée ;
-   les interventions peuvent être réalisées ;
-   les tests sont enregistrés ;
-   les pannes peuvent être clôturées selon les règles ;
-   les maintenances préventives sont suivies ;
-   les indicateurs sont calculés ;
-   les rôles sont respectés ;
-   les actions importantes sont auditées ;
-   les données sont sauvegardées ;
-   les tests critiques passent ;
-   l'application peut être déployée.

## 19. Évolution après MVP

Seulement après validation du MVP :

``` text
MVP
 ↓
Niveau 3 — Analyse
 ↓
Niveau 4 — Aide à la décision
 ↓
Niveau 5 — Analyse intelligente
```

Le scoring prédictif et les modèles statistiques/ML appartiennent aux
niveaux avancés et ne doivent pas contaminer le MVP.
