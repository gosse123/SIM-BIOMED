# SIM-BIOMED — ARCHITECTURE LOGICIELLE

## Système Intelligent de Maintenance Biomédicale

*Document d'architecture dérivé de la logique métier SIM-BIOMED*

---

# 1. Vue d'ensemble de l'architecture

SIM-BIOMED est conçu selon une **architecture en couches (layered architecture)**, organisée autour du domaine métier, afin que la logique de maintenance biomédicale (gammes, pannes, indicateurs, criticité) reste au centre du système.

```text
┌─────────────────────────────────────────────────────────────────┐
│                      COUCHE PRÉSENTATION                        │
│  Portail web interne  |  Application mobile  |  Tableaux de bord│
└──────────────────────────────┬──────────────────────────────────┘
                               │ HTTPS / REST / WebSocket
┌──────────────────────────────▼──────────────────────────────────┐
│                       COUCHE API / GATEWAY                      │
│  Authentification & autorisations  |  Routage  |  Validation    │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                     COUCHE SERVICES MÉTIER                      │
│  Gestion du parc  |  Gestion des pannes  |  Maintenance prév. │
│  Indicateurs (MTBF/MTTR)  |  Criticité  |  Priorisation       │
│  Moteur d'alertes  |  Moteur d'aide à la décision              │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                        COUCHE DONNÉES                           │
│  Base de données relationnelle  |  Journal d'événements (log)  │
│  Stockage documentaire  |  Entrepôt analytique (niveau 4-5)    │
└─────────────────────────────────────────────────────────────────┘
```

---

# 2. Principes directeurs d'architecture

1. **Le domaine métier est au centre** : les règles de criticité, de priorisation et de cycle de vie sont implémentées dans une couche métier dédiée, indépendante des interfaces.
2. **Traçabilité totale** : chaque événement (panne, intervention, transfert, réforme) laisse une trace horodatée dans l'historique de l'équipement (§4 et §5 du document métier).
3. **Évolutivité progressive** : les niveaux 1 à 5 (enregistrement → analyse intelligente) sont des briques activables successivement, sans refonte du socle.
4. **Indicateurs au service de la décision** : les calculs MTBF, MTTR, disponibilité ne sont pas de simples rapports mais alimentent la priorisation et les recommandations.
5. **Sécurité et conformité** : gestion fine des rôles, données sensibles chiffrées, audit complet des actions.

---

# 3. Couche présentation

## 3.1 Modules frontaux

| Module | Fonction | Public |
|---|---|---|
| **Portail web interne** | Consultation du parc, fiches équipements, gestion des interventions | Biomédical, services cliniques |
| **Application mobile / tablette** | Signalement de panne sur le terrain, scan de code-barres/QR de l'équipement | Personnel soignant, techniciens |
| **Tableaux de bord décisionnels** | KPIs, MTBF/MTTR, disponibilité, alertes de criticité | Direction, responsable biomédical |
| **Module technicien** | File de travail priorisée, diagnostic, clôture d'intervention | Techniciens biomédicaux |

## 3.2 Fonctionnalités clés par écran (alignées sur le processus métier §13)

- **Fiche équipement** : identité complète, statut opérationnel (coloré selon les 7 statuts), cycle de vie, historique chronologique.
- **Écran de signalement** : équipement (scan), date/heure automatique, service, déclarant, symptôme.
- **File d'interventions priorisées** : liste ordonnée selon la fonction de priorité (§10).
- **Tableau de bord des indicateurs** : MTBF, MTTR, disponibilité, taux de panne, avec tendances.
- **Écran de maintenance préventive** : équipements à jour / prochains / en retard (code couleur §7).

---

# 4. Couche API / Gateway

- **API REST** (ou GraphQL) exposant les ressources : `/equipements`, `/pannes`, `/interventions`, `/maintenances-preventives`, `/pieces`, `/techniciens`, `/prestataires`, `/indicateurs`, `/alertes`.
- **Authentification** : SSO de l'établissement (LDAP/AD ou OAuth2) + gestion des sessions.
- **Autorisation basée sur les rôles (RBAC)** :
  - *Administrateur* : configuration, paramétrage des criticités et périodicités.
  - *Responsable biomédical* : qualification, criticité, validation, clôture.
  - *Technicien* : diagnostic, intervention, tests.
  - *Personnel soignant* : signalement, consultation du statut de son service.
  - *Direction* : lecture des indicateurs et tableaux de bord.
- **Journalisation (audit log)** : chaque action sensible (changement de statut, clôture, réforme) est tracée avec utilisateur, horodatage et ancienne/nouvelle valeur.

---

# 5. Couche services métier

C'est le cœur du système. Chaque service correspond à un objet métier du document (§14).

## 5.1 Service Gestion du Parc

- CRUD équipements avec les 15 attributs (§2) : nom, type, catégorie, marque, modèle, n° série, n° inventaire, service, localisation, dates, état, criticité, statut cycle de vie.
- Identifiant unique par équipement (code-barres/QR).
- Gestion des événements administratifs : réception, installation, mise en service, transfert, changement de localisation/service, mise hors service, réforme.
- Mise à jour automatique du **statut opérationnel** selon les événements (§3).

## 5.2 Service Gestion des Pannes

Implémente la machine à états du §6 :

```text
SIGNALÉ → QUALIFIÉ → CRITICITÉ ÉVALUÉE → EN DIAGNOSTIC
   → EN INTERVENTION → EN TEST/CONTRÔLE → CLOS
                                    ↘ EN ATTENTE PIÈCE / PRESTATAIRE
                                    ↘ HORS SERVICE / RÉFORMÉ
```

Chaque transition est validée par des règles métier :
- La clôture exige un résultat de test (conforme, sous surveillance, non conforme, toujours en panne).
- La remise en service met à jour le statut opérationnel de l'équipement.
- Toutes les actions d'intervention (réglage, réparation, pièce, calibration, prestataire) sont enregistrées en historique.

## 5.3 Service Maintenance Préventive

- Planification par périodicité configurable (recommandations fabricant, criticité, heures d'usage, cycles, réglementation — §7).
- Calcul automatique des états : **à jour / prochaine / en retard** (avec seuils configurables).
- Génération automatique des ordres de travail préventifs.

## 5.4 Service Indicateurs

Moteur de calcul périodique et à la demande :

| Indicateur | Formule | Usage décisionnel |
|---|---|---|
| MTBF | temps de fonctionnement total / nb pannes | fiabilité, tendance |
| MTTR | temps total de réparation / nb réparations | organisation, approvisionnement |
| Disponibilité | MTBF / (MTBF + MTTR) | objectifs de service |
| Indisponibilité | 1 − disponibilité | rapport de performance |
| Taux de panne | nb pannes / temps de fonctionnement | analyse par parc |

Calculs disponibles par équipement, par catégorie, par service, sur période glissante.

## 5.5 Service Criticité

- Matrice de criticité paramétrable combinant : impact clinique, sécurité patient, disponibilité d'un remplacement, fréquence des pannes, difficulté de réparation, coût de l'indisponibilité (§9).
- Classification : critique / élevée / moyenne / faible.
- Recalcul automatique si l'historique des pannes évolue.

## 5.6 Service Priorisation

- Fonction configurable : `Priorité = f(criticité, gravité, urgence, impact soins, disponibilité remplacement)` (§10).
- Production d'une file de travail ordonnée pour les techniciens.
- Prise en compte du contexte : réaffectation si un respirateur tombe en panne.

## 5.7 Service Alertes et Notifications

- Alertes temps réel : panne critique, maintenance préventive en retard, équipement resté longtemps en attente de pièce.
- Canaux : notification in-app, e-mail, SMS pour les pannes critiques.

## 5.8 Service Aide à la Décision (niveaux 3 à 5)

- **Niveau 3 (Analyse)** : détection de tendances (baisse de MTBF, hausse du taux de panne).
- **Niveau 4 (Aide à la décision)** : recommandations explicites (« inspection anticipée recommandée », « révision de la périodicité »).
- **Niveau 5 (Analyse intelligente)** : modèle prédictif de risque de défaillance fondé sur l'historique (approche par scoring, puis modèles statistiques/apprentissage).

---

# 6. Couche données

## 6.1 Base de données relationnelle (système de gestion principal)

Principales entités (dérivées des objets métier §14) :

```text
EQUIPEMENT (id, nom, type, categorie, marque, modele, num_serie,
            num_inventaire, service_id, localisation_id,
            date_reception, date_installation, date_mise_service,
            etat_operationnel, niveau_criticite, statut_cycle_vie)

SERVICE_UTILISATEUR (id, nom, description)
LOCALISATION (id, batiment, etage, salle)

EVENEMENT (id, equipement_id, type [technique/maintenance/administratif],
           sous_type, date, auteur_id, description, donnees_json)

PANNE (id, equipement_id, date_signalement, declarant_id, service_id,
       symptome, nature, type_panne, gravite, urgence, impact_soins,
       criticite_evaluee, statut [signale/qualifie/diagnostic/
       intervention/test/clos/attente], priorite)

DIAGNOSTIC (id, panne_id, date, technicien_id, cause_probable,
            cause_confirmee, elements_defectueux)

INTERVENTION (id, panne_id, date_debut, date_fin, technicien_id,
              type [reglage/reparation/nettoyage/piece/calibration/prestataire],
              actions, resultat_test [conforme/surveillance/non_conforme/
              toujours_panne], statut_final)

PIECE_RECHANGE (id, reference, designation, stock, seuil_alerte)
INTERVENTION_PIECE (intervention_id, piece_id, quantite)

MAINTENANCE_PREVENTIVE (id, equipement_id, type, periodicite,
                        derniere_realisation, prochaine_echeance, etat)

TECHNICIEN (id, nom, competences[], disponibilite)
PRESTATAIRE (id, nom, specialite, contrat)

INDICATEUR (id, equipement_id, periode, mtbf, mttr, disponibilite,
            indisponibilite, taux_panne)

ALERTE (id, type, equipement_id, message, severite, date, statut [lue/traitee])

AUDIT_LOG (id, utilisateur_id, action, entite, entite_id,
           ancienne_valeur, nouvelle_valeur, horodatage)
```

## 6.2 Journal d'événements

Tous les événements (§5 du document métier : techniques, maintenance, administratifs) sont également écrits dans un journal append-only qui constitue l'historique complet et horodaté de chaque équipement. Source de vérité pour les indicateurs et l'analyse.

## 6.3 Entrepôt analytique (optionnel, niveaux 4-5)

Réplication périodique des données vers un stockage analytique pour les tendances long terme, le scoring prédictif et les tableaux de bord multi-établissements.

---

# 7. Processus transverses d'exécution

## 7.1 Déroulement d'une panne (implémentation du §13)

```text
1. Signalement (mobile/web) → création PANNE(statut = SIGNALÉ)
2. Qualification → PANNE(statut = QUALIFIÉ) + gravité/urgence/impact
3. Évaluation criticité → criticité_evaluee recalculée
4. Priorisation → insertion ordonnée dans la file des techniciens
5. Diagnostic → enregistrement cause probable/confirmée
6. Intervention → actions + pièces consommées tracées
7. Test/contrôle → résultat enregistré (bloquant pour la clôture)
8. Clôture ou poursuite → statut opérationnel équipement mis à jour
9. Historique + indicateurs recalculés
10. Moteur d'analyse → alertes ou recommandations éventuelles
```

## 7.2 Calcul nocturne (job planifié)

- Mise à jour des échéances de maintenance préventive et des états à jour/prochaine/en retard.
- Recalcul des indicateurs (MTBF, MTTR, disponibilité, taux de panne).
- Recalcul des criticités et réévaluation des priorités.
- Détection de tendances et génération de recommandations (niveaux 3-5).

---

# 8. Exigences non fonctionnelles

| Domaine | Exigence |
|---|---|
| **Disponibilité** | Service interne critique : objectif ≥ 99,5 %, fonctionnement dégradé possible en mode hors ligne pour le signalement mobile (synchronisation différée) |
| **Performance** | Fiche équipement < 2 s ; tableau de bord indicateurs < 5 s sur 10 000 équipements |
| **Sécurité** | Chiffrement TLS en transit, chiffrement au repos des données sensibles, RBAC, audit complet |
| **Conformité** | Traçabilité compatible exigences qualité/réglementaires (marquage, historique d'interventions) |
| **Sauvegarde** | Sauvegardes quotidiennes, journal d'événements répliqué, plan de reprise |
| **Interfaçage** | API ouverte pour intégration éventuelle (gestion des stocks, achats, dossier patient) |

---

# 9. Feuille de route d'implémentation par niveaux

| Phase | Contenu | Niveau métier |
|---|---|---|
| **P1** | Parc, statuts opérationnels, événements, signalement et suivi de panne | Niveau 1 — Enregistrement |
| **P2** | Criticité, gravité, file priorisée, alertes | Niveau 2 — Organisation |
| **P3** | Maintenance préventive planifiée, indicateurs MTBF/MTTR/disponibilité, tableaux de bord | Niveau 3 — Analyse |
| **P4** | Recommandations (inspection anticipée, révision périodicité), analyse de tendances | Niveau 4 — Aide à la décision |
| **P5** | Scoring prédictif du risque de défaillance | Niveau 5 — Analyse intelligente |

---

# 10. Schéma de déploiement (cible)

```text
                ┌─────────────┐
                │  Navigateur  │      ┌──────────────┐
                │  (portail)   │      │   Mobile     │
                └──────┬───────┘      └──────┬───────┘
                       │ HTTPS             │
                ┌──────▼───────────────────▼───────┐
                │   Reverse proxy / API Gateway    │
                │   (auth, RBAC, rate limiting)    │
                └──────┬───────────────────────────┘
        ┌──────────────┼──────────────┐
┌───────▼──────┐ ┌─────▼──────┐ ┌─────▼──────────┐
│  Application │ │ Application│ │  Moteur jobs   │
│  métier      │ │ mobile API │ │  (indicateurs, │
│  (web)       │ │            │ │  alertes, ML)  │
└───────┬──────┘ └─────┬──────┘ └─────┬──────────┘
        └──────────────┼──────────────┘
                ┌──────▼────────────────┐
                │  Base de données      │
                │  relationnelle +      │
                │  journal d'événements │
                └───────────────────────┘
```

---

# 11. Alignement avec la vision finale

L'architecture matérialise la chaîne de valeur du §15 du document métier :

```text
Données terrain  → modules de saisie (web/mobile, scan équipement)
Historique     → journal d'événements + tables d'historique
Analyse        → moteur d'indicateurs + jobs de tendances
Indicateurs    → service indicateurs (MTBF, MTTR, disponibilité…)
Risques        → service criticité + scoring (niveau 5)
Priorisation   → service de priorisation + file de travail
Décision       → tableaux de bord + recommandations + alertes
Amélioration   → retour terrain : interventions, retours d'expérience
```
