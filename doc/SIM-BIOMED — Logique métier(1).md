# SIM-BIOMED — LOGIQUE MÉTIER

## Système Intelligent de Maintenance Biomédicale

---

# 1. La mission principale de SIM-BIOMED

SIM-BIOMED est une solution destinée à aider les établissements de santé à mieux organiser, suivre, analyser et améliorer la maintenance de leurs équipements biomédicaux.

Avant toute considération informatique ou technologique, SIM-BIOMED repose sur une question fondamentale :

> **Comment organiser les informations et les activités de maintenance afin d'aider les professionnels biomédicaux à prendre de meilleures décisions ?**

Le système doit principalement permettre de répondre à cinq grandes questions :

1. Quels équipements possédons-nous ?
2. Quel est l'état actuel de chaque équipement ?
3. Quels problèmes ou pannes ont été rencontrés ?
4. Quelles actions de maintenance ont été réalisées ou doivent être réalisées ?
5. Quelles décisions doivent être prises pour améliorer la disponibilité et la fiabilité des équipements ?

---

# 2. Gestion du parc biomédical

SIM-BIOMED doit permettre d'avoir une vision claire et actualisée de l'ensemble du parc biomédical.

Chaque équipement doit posséder une identité propre.

## Informations principales

Pour chaque équipement, le système doit pouvoir enregistrer :

- Nom de l'équipement
- Type d'équipement
- Catégorie
- Marque
- Modèle
- Numéro de série
- Numéro d'inventaire
- Service utilisateur
- Localisation
- Date de réception
- Date d'installation
- Date de mise en service
- État actuel
- Niveau de criticité
- Statut dans le cycle de vie

L'objectif est que chaque équipement puisse être identifié et suivi individuellement.

---

# 3. État opérationnel de l'équipement

Chaque équipement doit posséder un statut opérationnel clairement défini.

Les statuts possibles peuvent être :

- 🟢 **Fonctionnel**
- 🟠 **Fonctionnel sous surveillance**
- 🔴 **En panne**
- 🔵 **En maintenance**
- 🟣 **En attente de pièce ou de prestataire**
- ⚫ **Hors service**
- ⚪ **Réformé**

Le statut doit évoluer en fonction des événements enregistrés.

---

# 4. Cycle de vie d'un équipement biomédical

SIM-BIOMED doit suivre l'équipement tout au long de son cycle de vie.

```text
Réception
    ↓
Installation
    ↓
Mise en service
    ↓
Exploitation
    ↓
Maintenance préventive
    ↓
Panne éventuelle
    ↓
Maintenance corrective
    ↓
Test et contrôle
    ↓
Remise en service
    ↓
Réforme éventuelle
```

Chaque étape importante doit laisser une trace dans l'historique de l'équipement.

---

# 5. Gestion des événements

Un équipement biomédical évolue à travers différents événements.

SIM-BIOMED doit permettre d'enregistrer ces événements afin de constituer l'historique technique de l'équipement.

## 5.1 Événements techniques

- Panne
- Dysfonctionnement
- Alarme
- Anomalie
- Contrôle
- Inspection
- Dégradation des performances

## 5.2 Événements de maintenance

- Maintenance préventive
- Maintenance corrective
- Maintenance améliorative
- Inspection technique
- Vérification
- Calibration
- Réparation
- Remplacement d'une pièce
- Nettoyage technique

## 5.3 Événements administratifs

- Réception
- Installation
- Mise en service
- Transfert
- Changement de localisation
- Changement de service
- Mise hors service
- Réforme

---

# 6. Logique de gestion d'une panne

Lorsqu'un problème est constaté sur un équipement, SIM-BIOMED doit suivre une logique précise.

## Étape 1 : Signalement

Le problème est signalé.

Exemple :

> Le moniteur multiparamétrique ne s'allume plus.

Le signalement doit identifier :

- L'équipement concerné
- La date et l'heure
- Le service
- Le déclarant
- Le symptôme observé

---

## Étape 2 : Qualification

Le problème doit être analysé afin de déterminer :

- La nature du problème
- Le type de panne
- Le niveau de gravité
- L'impact sur les soins
- Le niveau d'urgence

---

## Étape 3 : Évaluation de la criticité

Toutes les pannes ne doivent pas être traitées avec la même priorité.

Par exemple, une panne sur un respirateur utilisé en réanimation peut avoir des conséquences plus importantes qu'une panne sur un équipement non critique disposant d'un appareil de remplacement.

La criticité doit donc être prise en compte.

Les niveaux peuvent être :

| Niveau | Signification |
|---|---|
| Critique | Risque important pour la continuité des soins ou la sécurité |
| Élevé | Service fortement perturbé |
| Moyen | Fonctionnement dégradé |
| Faible | Impact limité |

---

## Étape 4 : Diagnostic

Le technicien ou l'équipe biomédicale réalise une investigation afin de déterminer :

- La cause probable
- La cause confirmée
- Les éléments défectueux
- Les actions nécessaires

---

## Étape 5 : Intervention

L'intervention peut comprendre :

- Réglage
- Réparation
- Nettoyage
- Remplacement d'une pièce
- Calibration
- Intervention d'un prestataire externe

Toutes les actions doivent être enregistrées.

---

## Étape 6 : Test et contrôle

Après intervention, l'équipement doit être contrôlé afin de vérifier son bon fonctionnement.

Le résultat peut être :

- Conforme
- Fonctionnel sous surveillance
- Non conforme
- Toujours en panne

---

## Étape 7 : Clôture ou poursuite

L'intervention peut être clôturée uniquement lorsque le résultat final est connu.

L'équipement peut alors :

- Être remis en service
- Rester sous surveillance
- Rester en panne
- Attendre une pièce
- Attendre un prestataire
- Être mis hors service

---

# 7. Logique de la maintenance préventive

SIM-BIOMED doit aider à répondre à la question :

> **Quand faut-il intervenir avant qu'une panne ne survienne ?**

La périodicité de maintenance peut dépendre de plusieurs facteurs :

- Recommandations du fabricant
- Niveau de criticité
- Temps d'utilisation
- Nombre de cycles
- Environnement d'utilisation
- Historique des pannes
- Résultats des inspections
- Exigences réglementaires
- Politique de maintenance de l'établissement

---

## États de la maintenance préventive

Un équipement peut être :

- 🟢 **À jour**
- 🟠 **Maintenance prochaine**
- 🔴 **Maintenance en retard**

À long terme, SIM-BIOMED pourra également recommander une intervention anticipée.

Exemple :

> La fréquence des pannes de cet équipement augmente. Une inspection anticipée est recommandée.

---

# 8. Logique des indicateurs de maintenance

Les indicateurs doivent permettre d'interpréter la situation réelle du parc biomédical.

Ils ne doivent pas être considérés comme de simples résultats mathématiques.

Ils doivent servir à prendre des décisions.

---

## 8.1 MTBF — Temps moyen entre deux pannes

Question métier :

> **Combien de temps l'équipement fonctionne-t-il en moyenne avant de subir une panne ?**

Formule :

```text
MTBF = Temps total de fonctionnement / Nombre de pannes
```

Interprétation :

Un MTBF élevé peut indiquer une meilleure fiabilité.

Une baisse du MTBF peut indiquer une augmentation de la fréquence des défaillances.

---

## 8.2 MTTR — Temps moyen de réparation

Question métier :

> **Combien de temps faut-il en moyenne pour remettre l'équipement en état ?**

Formule :

```text
MTTR = Temps total de réparation / Nombre de réparations
```

Un MTTR élevé peut être lié à :

- Une difficulté de diagnostic
- Un manque de pièces de rechange
- L'absence de technicien spécialisé
- L'attente d'un prestataire
- Des difficultés d'approvisionnement

---

## 8.3 Disponibilité

Question métier :

> **Pendant combien de temps l'équipement est-il réellement disponible pour les soins ?**

Formule simplifiée :

```text
Disponibilité = MTBF / (MTBF + MTTR)
```

La disponibilité peut être exprimée en pourcentage.

---

## 8.4 Indisponibilité

Question métier :

> **Pendant combien de temps l'équipement n'est-il pas disponible pour assurer sa fonction ?**

Formule :

```text
Indisponibilité = 1 - Disponibilité
```

ou :

```text
Indisponibilité (%) = 100 - Disponibilité (%)
```

---

## 8.5 Taux de panne

Question métier :

> **À quelle fréquence les défaillances apparaissent-elles ?**

Formule :

```text
Taux de panne = Nombre de pannes / Temps de fonctionnement
```

---

# 9. Logique de criticité

SIM-BIOMED doit aider à identifier les équipements nécessitant une attention particulière.

La criticité peut être déterminée à partir de plusieurs critères.

## Critères possibles

### Impact clinique

Quelle est la conséquence de l'indisponibilité de l'équipement sur la prise en charge du patient ?

### Sécurité du patient

Une défaillance peut-elle présenter un risque pour le patient ou l'utilisateur ?

### Disponibilité d'un équipement de remplacement

Existe-t-il une solution de secours ?

### Fréquence des pannes

L'équipement connaît-il régulièrement des défaillances ?

### Difficulté de réparation

La réparation nécessite-t-elle des compétences, des pièces ou un prestataire spécialisé ?

### Coût de l'indisponibilité

L'arrêt de l'équipement entraîne-t-il une conséquence importante pour le service ?

---

## Classification possible

- 🔴 Critique
- 🟠 Élevée
- 🟡 Moyenne
- 🟢 Faible

---

# 10. Logique de priorisation des interventions

SIM-BIOMED ne doit pas uniquement afficher une liste des équipements en panne.

Le système doit aider à déterminer :

> **Quel équipement doit être traité en priorité ?**

La priorité peut prendre en compte :

- La criticité de l'équipement
- La gravité de la panne
- L'urgence
- L'impact sur les soins
- La disponibilité d'un équipement de remplacement

Exemple de logique :

```text
Priorité = Fonction de la criticité + gravité + urgence + impact
```

Exemple :

| Équipement | Criticité | Gravité | Urgence | Priorité |
|---|---|---|---|---|
| Respirateur | Très élevée | Élevée | Élevée | 1 |
| Moniteur | Élevée | Moyenne | Moyenne | 2 |
| Pèse-bébé | Faible | Faible | Faible | 3 |

---

# 11. Niveaux d'évolution de SIM-BIOMED

SIM-BIOMED peut évoluer progressivement.

## Niveau 1 — Enregistrement

Le système permet de savoir :

> Une panne est enregistrée.

---

## Niveau 2 — Organisation

Le système aide à savoir :

> Cette panne doit être traitée en priorité.

---

## Niveau 3 — Analyse

Le système permet de constater :

> Cet équipement présente une augmentation de sa fréquence de panne.

---

## Niveau 4 — Aide à la décision

Le système recommande :

> Une maintenance approfondie est recommandée.

---

## Niveau 5 — Analyse intelligente

Le système peut progressivement détecter :

> Sur la base de l'historique et des tendances observées, le risque de défaillance augmente.

---

# 12. Le cœur métier de SIM-BIOMED

Le cœur de SIM-BIOMED n'est pas simplement de calculer un MTBF ou un MTTR.

L'objectif principal est :

> **Aider les professionnels biomédicaux à prendre de meilleures décisions grâce aux données de maintenance.**

Les indicateurs tels que :

- MTBF
- MTTR
- Disponibilité
- Indisponibilité
- Taux de panne

sont des outils permettant d'interpréter l'état réel des équipements et d'orienter les décisions.

---

# 13. Processus métier global

Le fonctionnement général d'un équipement dans SIM-BIOMED peut être représenté ainsi :

```text
Équipement
    ↓
Exploitation
    ↓
Surveillance
    ↓
Événement détecté
    ↓
Signalement
    ↓
Qualification
    ↓
Évaluation de la criticité
    ↓
Priorisation
    ↓
Diagnostic
    ↓
Intervention
    ↓
Test et contrôle
    ↓
Remise en service ou poursuite de la maintenance
    ↓
Mise à jour de l'historique
    ↓
Calcul des indicateurs
    ↓
Analyse
    ↓
Aide à la décision
```

---

# 14. Objets métier principaux

Les principaux objets métier de SIM-BIOMED sont :

1. **Équipement biomédical**
2. **Service utilisateur**
3. **Localisation**
4. **Panne**
5. **Signalement**
6. **Diagnostic**
7. **Intervention**
8. **Maintenance préventive**
9. **Maintenance corrective**
10. **Pièce de rechange**
11. **Technicien**
12. **Prestataire**
13. **Indicateur de maintenance**
14. **Alerte**
15. **Niveau de criticité**
16. **Priorité**

---

# 15. Vision finale

La logique métier de SIM-BIOMED repose sur le principe suivant :

```text
DONNÉES TERRAIN
      ↓
HISTORIQUE
      ↓
ANALYSE
      ↓
INDICATEURS
      ↓
ÉVALUATION DES RISQUES
      ↓
PRIORISATION
      ↓
AIDE À LA DÉCISION
      ↓
AMÉLIORATION DE LA MAINTENANCE
```

SIM-BIOMED doit donc progressivement devenir un système permettant de transformer les données issues du terrain en informations utiles, puis en décisions permettant d'améliorer la disponibilité, la fiabilité et la gestion des équipements biomédicaux.