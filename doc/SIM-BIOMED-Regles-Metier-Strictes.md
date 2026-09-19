# SIM-BIOMED --- Règles métier strictes

## 1. Objet

Ce document constitue la référence normative du comportement métier de
SIM-BIOMED.

Une fonctionnalité, une interface ou une implémentation technique ne
doit pas contredire ces règles.

En cas de doute :

``` text
Règles métier
      >
Architecture technique
      >
API
      >
Interface utilisateur
```

L'interface doit représenter le métier. Elle ne doit jamais définir
seule le métier.

## 2. Principes non négociables

### RB-001 --- Le système doit conserver la traçabilité

Tout événement important concernant un équipement doit laisser une
trace.

### RB-002 --- Le domaine métier est indépendant de l'interface

Une règle métier doit être applicable que l'action provienne du web,
d'une future application mobile ou d'une autre API autorisée.

### RB-003 --- Le serveur est l'autorité

Une règle métier ne doit jamais être appliquée uniquement dans le
frontend.

### RB-004 --- Pas de transition implicite

Un changement d'état doit correspondre à une transition métier
autorisée.

### RB-005 --- Pas de clôture sans résultat

Une panne ne peut être clôturée que lorsque le résultat du test/contrôle
est connu.

### RB-006 --- Toute action sensible est traçable

Les changements importants doivent enregistrer utilisateur, date/heure,
ancienne valeur et nouvelle valeur lorsque ces informations sont
pertinentes.

------------------------------------------------------------------------

# 3. Équipement

## RB-EQ-001 --- Identité unique

Chaque équipement possède un identifiant unique.

Cet identifiant ne doit jamais être réutilisé pour un autre équipement.

## RB-EQ-002 --- Identification

Un équipement doit pouvoir être identifié individuellement.

Les informations prévues sont notamment :

-   nom ;
-   type ;
-   catégorie ;
-   marque ;
-   modèle ;
-   numéro de série ;
-   numéro d'inventaire ;
-   service ;
-   localisation ;
-   dates importantes ;
-   état ;
-   criticité ;
-   statut du cycle de vie.

## RB-EQ-003 --- Statut opérationnel

Un équipement possède un seul statut opérationnel courant.

Valeurs autorisées :

``` text
FONCTIONNEL
FONCTIONNEL_SOUS_SURVEILLANCE
EN_PANNE
EN_MAINTENANCE
EN_ATTENTE_PIECE_OU_PRESTATAIRE
HORS_SERVICE
REFORME
```

## RB-EQ-004 --- Réformé

Un équipement réformé ne peut plus être considéré comme opérationnel.

## RB-EQ-005 --- Historique

Les changements importants du statut opérationnel doivent être
historisés.

------------------------------------------------------------------------

# 4. Cycle de vie

## RB-LC-001 --- Ordre général

Le cycle de vie de référence est :

``` text
Réception
  ↓
Installation
  ↓
Mise en service
  ↓
Exploitation
  ↓
Maintenance
  ↓
Réforme éventuelle
```

Le système doit conserver les événements correspondant aux étapes
importantes.

## RB-LC-002 --- Pas de suppression historique

Les événements historiques ne doivent pas être supprimés pour masquer
une action passée.

Une correction doit elle-même être traçable lorsqu'elle modifie une
donnée métier importante.

------------------------------------------------------------------------

# 5. Événements

## RB-EV-001 --- Trois familles

Les événements appartiennent à trois familles :

``` text
TECHNIQUE
MAINTENANCE
ADMINISTRATIF
```

## RB-EV-002 --- Événement technique

Peut notamment représenter :

-   panne ;
-   dysfonctionnement ;
-   alarme ;
-   anomalie ;
-   contrôle ;
-   inspection ;
-   dégradation des performances.

## RB-EV-003 --- Événement maintenance

Peut notamment représenter :

-   maintenance préventive ;
-   maintenance corrective ;
-   maintenance améliorative ;
-   inspection ;
-   vérification ;
-   calibration ;
-   réparation ;
-   remplacement de pièce ;
-   nettoyage technique.

## RB-EV-004 --- Événement administratif

Peut notamment représenter :

-   réception ;
-   installation ;
-   mise en service ;
-   transfert ;
-   changement de localisation ;
-   changement de service ;
-   mise hors service ;
-   réforme.

------------------------------------------------------------------------

# 6. Signalement d'une panne

## RB-PN-001 --- Création

Un signalement doit identifier au minimum :

-   équipement ;
-   date/heure ;
-   service ;
-   déclarant ;
-   symptôme observé.

## RB-PN-002 --- Statut initial

Toute nouvelle panne commence à :

``` text
SIGNALEE
```

## RB-PN-003 --- Pas de qualification automatique arbitraire

Le système ne doit pas inventer la nature, la gravité ou l'urgence d'une
panne à partir d'informations insuffisantes.

Ces informations doivent être fournies ou déterminées selon les règles
métier prévues.

------------------------------------------------------------------------

# 7. Qualification

## RB-PN-004 --- Informations de qualification

La qualification détermine :

-   nature du problème ;
-   type de panne ;
-   gravité ;
-   impact sur les soins ;
-   urgence.

## RB-PN-005 --- Transition

Une panne qualifiée passe à :

``` text
QUALIFIEE
```

Les informations nécessaires doivent être présentes avant cette
transition.

------------------------------------------------------------------------

# 8. Criticité

## RB-CR-001 --- Niveaux

Les niveaux sont :

``` text
CRITIQUE
ELEVE
MOYEN
FAIBLE
```

## RB-CR-002 --- Critères

La criticité peut prendre en compte :

-   impact clinique ;
-   sécurité du patient ;
-   disponibilité d'un équipement de remplacement ;
-   fréquence des pannes ;
-   difficulté de réparation ;
-   coût de l'indisponibilité.

## RB-CR-003 --- Pas de criticité arbitraire

Une criticité ne doit pas être déterminée simplement parce qu'un
équipement est coûteux ou important financièrement.

Elle doit respecter les critères métier définis.

## RB-CR-004 --- Recalcul

La criticité peut être réévaluée lorsque les données pertinentes
évoluent.

Toute modification importante doit être traçable.

------------------------------------------------------------------------

# 9. Priorisation

## RB-PR-001 --- Objectif

La priorité sert à déterminer l'ordre dans lequel les interventions
doivent être traitées.

## RB-PR-002 --- Facteurs

La priorité prend en compte :

-   criticité ;
-   gravité ;
-   urgence ;
-   impact sur les soins ;
-   disponibilité d'un remplacement.

## RB-PR-003 --- Priorité ≠ criticité

La criticité et la priorité sont deux notions différentes.

``` text
Criticité = importance du risque / impact de l'équipement

Priorité = ordre de traitement d'une situation donnée
```

## RB-PR-004 --- File de travail

Le système doit pouvoir produire une file d'interventions ordonnée.

La file doit être recalculable lorsque les informations influençant la
priorité changent.

------------------------------------------------------------------------

# 10. Diagnostic

## RB-DI-001 --- Diagnostic

Le diagnostic peut enregistrer :

-   cause probable ;
-   cause confirmée ;
-   éléments défectueux ;
-   actions nécessaires.

## RB-DI-002 --- Pas de faux diagnostic

Le système ne doit jamais présenter une cause probable comme une cause
confirmée.

------------------------------------------------------------------------

# 11. Intervention

## RB-IN-001 --- Traçabilité

Une intervention doit conserver au minimum les informations nécessaires
pour savoir :

-   quand elle a commencé ;
-   quand elle s'est terminée ;
-   qui l'a réalisée ;
-   quelles actions ont été effectuées ;
-   quel type d'intervention a été réalisé.

## RB-IN-002 --- Types

Les types peuvent inclure :

``` text
REGLAGE
REPARATION
NETTOYAGE
REMPLACEMENT_PIECE
CALIBRATION
PRESTATAIRE
```

## RB-IN-003 --- Pièces

Lorsqu'une pièce est consommée, la consommation doit être enregistrée.

------------------------------------------------------------------------

# 12. Test et contrôle

## RB-TEST-001 --- Test obligatoire avant clôture

Une panne ne peut pas passer à `CLOSE` sans résultat de test/contrôle.

## RB-TEST-002 --- Résultats autorisés

``` text
CONFORME
SOUS_SURVEILLANCE
NON_CONFORME
TOUJOURS_EN_PANNE
```

## RB-TEST-003 --- Résultat conforme

Un résultat conforme permet la remise en service lorsque les autres
conditions métier sont satisfaites.

## RB-TEST-004 --- Sous surveillance

Un équipement fonctionnel sous surveillance ne doit pas être présenté
comme totalement normal.

## RB-TEST-005 --- Non conforme

Un résultat non conforme interdit la clôture normale comme intervention
réussie.

## RB-TEST-006 --- Toujours en panne

Le système doit conserver la situation de panne et poursuivre le
traitement approprié.

------------------------------------------------------------------------

# 13. Clôture

## RB-CL-001 --- Condition de clôture

La clôture exige :

1.  une intervention ou une action de maintenance lorsque nécessaire ;
2.  un résultat de test connu ;
3.  une décision finale cohérente avec ce résultat.

## RB-CL-002 --- Mise à jour équipement

La clôture ou la poursuite de la maintenance doit entraîner la mise à
jour cohérente du statut opérationnel de l'équipement.

## RB-CL-003 --- Pas de clôture administrative

Un utilisateur ne doit pas pouvoir clôturer une panne uniquement pour la
faire disparaître de sa file de travail.

------------------------------------------------------------------------

# 14. Attente de pièce / prestataire

## RB-AT-001

Une panne peut être placée en attente lorsqu'une pièce ou un prestataire
est nécessaire.

## RB-AT-002

L'attente ne signifie pas que la panne est résolue.

## RB-AT-003

Un équipement en attente de pièce ou de prestataire doit conserver un
statut opérationnel cohérent avec son état réel.

------------------------------------------------------------------------

# 15. Maintenance préventive

## RB-MP-001 --- Périodicité

La périodicité peut dépendre notamment :

-   recommandations du fabricant ;
-   criticité ;
-   temps d'utilisation ;
-   nombre de cycles ;
-   environnement ;
-   historique des pannes ;
-   inspections ;
-   exigences réglementaires ;
-   politique de l'établissement.

## RB-MP-002 --- États

``` text
A_JOUR
PROCHAINE
EN_RETARD
```

## RB-MP-003 --- Échéance

Une maintenance possède une prochaine échéance calculable.

## RB-MP-004 --- Retard

Une maintenance dont l'échéance est dépassée et qui n'est pas réalisée
doit être considérée comme en retard selon les seuils configurés.

------------------------------------------------------------------------

# 16. Indicateurs

## RB-IND-001 --- MTBF

``` text
MTBF =
Temps total de fonctionnement
/
Nombre de pannes
```

## RB-IND-002 --- MTTR

``` text
MTTR =
Temps total de réparation
/
Nombre de réparations
```

## RB-IND-003 --- Disponibilité

``` text
Disponibilité =
MTBF
/
(MTBF + MTTR)
```

## RB-IND-004 --- Indisponibilité

``` text
Indisponibilité = 1 - Disponibilité
```

## RB-IND-005 --- Taux de panne

``` text
Taux de panne =
Nombre de pannes
/
Temps de fonctionnement
```

## RB-IND-006 --- Interprétation

Les indicateurs ne doivent pas être traités comme de simples nombres
isolés.

Ils servent à analyser l'état du parc et à soutenir les décisions.

------------------------------------------------------------------------

# 17. Aide à la décision

## RB-AI-001 --- MVP

Le MVP ne doit pas prétendre prédire les pannes.

Il fournit :

-   données ;
-   historique ;
-   indicateurs ;
-   criticité ;
-   priorisation ;
-   alertes simples.

## RB-AI-002 --- Niveaux avancés

Les recommandations et la prédiction appartiennent aux évolutions
ultérieures.

``` text
Niveau 1 : Enregistrement
Niveau 2 : Organisation
Niveau 3 : Analyse
Niveau 4 : Aide à la décision
Niveau 5 : Analyse intelligente
```

## RB-AI-003 --- Pas de ML prématuré

Aucun modèle de machine learning ne doit être ajouté au MVP simplement
pour donner une apparence "intelligente" au produit.

------------------------------------------------------------------------

# 18. Rôles et autorisations

## RB-SEC-001 --- Administrateur

Peut notamment :

-   configurer le système ;
-   gérer les utilisateurs ;
-   paramétrer les règles autorisées.

## RB-SEC-002 --- Responsable biomédical

Peut notamment :

-   qualifier ;
-   évaluer la criticité ;
-   valider ;
-   clôturer.

## RB-SEC-003 --- Technicien

Peut notamment :

-   diagnostiquer ;
-   intervenir ;
-   enregistrer les tests.

## RB-SEC-004 --- Personnel soignant

Peut notamment :

-   signaler une panne ;
-   consulter les informations autorisées de son service.

## RB-SEC-005 --- Direction

Accès principalement orienté vers :

-   indicateurs ;
-   tableaux de bord ;
-   suivi global.

## RB-SEC-006 --- Sécurité serveur

Les permissions doivent être contrôlées côté backend.

------------------------------------------------------------------------

# 19. Audit

## RB-AUD-001

Les actions sensibles doivent être auditées.

## RB-AUD-002

Un audit important doit permettre de retrouver :

``` text
Utilisateur
Action
Entité
Identifiant de l'entité
Ancienne valeur
Nouvelle valeur
Date/heure
```

## RB-AUD-003

L'audit ne doit pas être modifiable par un utilisateur ordinaire.

------------------------------------------------------------------------

# 20. Règles anti-divagation du projet

Ces règles sont destinées à empêcher l'équipe de sortir du périmètre du
MVP.

## RB-SCOPE-001

Toute nouvelle fonctionnalité doit répondre à un besoin métier
identifié.

## RB-SCOPE-002

Une technologie ne justifie pas à elle seule une nouvelle
fonctionnalité.

## RB-SCOPE-003

Aucun microservice dans le MVP sans problème concret que le monolithe ne
peut raisonnablement résoudre.

## RB-SCOPE-004

Aucun système ML dans le MVP.

## RB-SCOPE-005

Aucune fonctionnalité de gestion du dossier patient dans le MVP.

## RB-SCOPE-006

Aucune fonctionnalité SMS obligatoire dans le MVP.

## RB-SCOPE-007

Aucune application mobile native obligatoire dans le MVP.

Une interface web responsive/PWA peut couvrir le besoin initial de
terrain.

## RB-SCOPE-008

Aucun Kubernetes dans le MVP.

## RB-SCOPE-009

Aucune optimisation de performance prématurée sans mesure montrant un
problème réel.

## RB-SCOPE-010

Aucune modification majeure du modèle métier sans justification
documentée.

------------------------------------------------------------------------

# 21. Règle de changement

Toute modification du métier doit suivre :

``` text
Problème identifié
      ↓
Justification
      ↓
Modification de la règle métier
      ↓
Mise à jour du document
      ↓
Mise à jour des tests
      ↓
Implémentation
```

Le code ne doit jamais devenir la seule source de vérité.

------------------------------------------------------------------------

# 22. Tests métier obligatoires

Chaque règle critique doit posséder au moins un test.

Priorité absolue :

1.  transitions de panne ;
2.  permissions ;
3.  clôture ;
4.  criticité ;
5.  priorité ;
6.  statut équipement ;
7.  calculs d'indicateurs ;
8.  maintenance préventive ;
9.  audit.

Exemple :

``` text
TEST :
Une panne sans résultat de test
        ↓
Tentative de clôture
        ↓
REFUS
```

Autre exemple :

``` text
TEST :
Résultat = CONFORME
        +
conditions satisfaites
        ↓
Clôture autorisée
```

------------------------------------------------------------------------

# 23. Règle finale

SIM-BIOMED ne doit pas chercher à "faire de l'IA" avant de maîtriser son
domaine.

La progression obligatoire est :

``` text
DONNÉES FIABLES
      ↓
HISTORIQUE FIABLE
      ↓
RÈGLES MÉTIER FIABLES
      ↓
INDICATEURS FIABLES
      ↓
PRIORISATION
      ↓
AIDE À LA DÉCISION
      ↓
ANALYSE INTELLIGENTE
```

La qualité des données et des règles métier est donc une condition
préalable à toute intelligence avancée.
