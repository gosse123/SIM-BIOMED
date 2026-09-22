# SIM-BIOMED — Guide d'Implémentation UI/UX, Architecture Frontend & Recette Qualité

> **Système Intelligent de Maintenance Biomédicale (GMAO Clinique)**  
> Référentiel technique et ergonomique pour l'implémentation frontend (React 18+, TypeScript, Tailwind CSS) et la validation conforme aux exigences hospitalières (HDS, PGSSI-S, ISO 13485).

---

## Sommaire
1. [Vision Produit & Principes Ergonomiques](#1-vision-produit--principes-ergonomiques)
2. [Design System & Fondations Globales](#2-design-system--fondations-globales)
3. [Spécifications Écran par Écran](#3-spécifications-écran-par-écran)
   - [A1 — Connexion & Authentification biomédicale](#a1--connexion--authentification-biomédicale)
   - [B1 — Tableau de bord général (Supervision)](#b1--tableau-de-bord-général-supervision)
   - [C1 — Gestion du parc biomédical (Inventaire)](#c1--gestion-du-parc-biomédical-inventaire)
   - [C3 — Fiche équipement détaillée (Traçabilité & Métrologie)](#c3--fiche-équipement-détaillée-traçabilité--métrologie)
   - [E1 — File d'interventions prioritaires (Régulation technique)](#e1--file-dinterventions-prioritaires-régulation-technique)
4. [Responsive Design & Adaptabilité Multi-Devices](#4-responsive-design--adaptabilité-multi-devices)
5. [Système d'Animations & Micro-Interactions](#5-système-danimations--micro-interactions)
6. [Bibliothèque de Composants Recommandée](#6-bibliothèque-de-composants-recommandée)
7. [Stratégie de Test & Protocole de Conformité Réglementaire](#7-stratégie-de-test--protocole-de-conformité-réglementaire)

---

## 1. Vision Produit & Principes Ergonomiques

SIM-BIOMED est un outil critique d'exploitation hospitalière. L'erreur de saisie ou le retard d'interprétation visuelle peut avoir un impact direct sur la continuité des soins et la sécurité des patients.

### Principes directeurs :
- **Clarté > Décoration** : Élimination du glassmorphism, des dégradés intempestifs et des animations superflues.
- **Règle des 3 Questions** : Tout écran doit permettre à l'utilisateur de répondre en moins de 3 secondes à :
  1. *Quel est l'état actuel de l'équipement ou du parc ?*
  2. *Pourquoi est-il dans cet état (cause, blocage, criticité) ?*
  3. *Quelle est la prochaine action autorisée par mon rôle ?*
- **Double encodage d'information** : Ne jamais signifier un statut uniquement par la couleur (ex: point rouge accompagné systématiquement du texte "Critique" ou "En panne" + icône dédiée).
- **Densité adaptée au poste** : Densité d'information optimisée sur poste fixe (moniteur atelier 1080p/2K) et aération tactile pour les tablettes en chambre de réanimation ou blocs opératoires.

---

## 2. Design System & Fondations Globales

### Palette Chromatique (Tokens Tailwind)

| Rôle | Token / Hex | Usage strict |
| :--- | :--- | :--- |
| **Fond Navigation** | `#0f172a` (`slate-900`) | Barre latérale persistante, contrastes structurels |
| **Surface Application** | `#f8f9ff` (`slate-50`) | Fond de page global clinique |
| **Surface Conteneur** | `#ffffff` (`white`) | Cartes KPI, modales, volets de données |
| **Bleu Médical Principal** | `#0284c7` (`sky-600`) | Boutons d'action prioritaires, filtres actifs, focus |
| **Bleu Secondaire / Hover**| `#0369a1` (`sky-700`) | Survol des boutons, états actifs |
| **Vert Conforme / Opérationnel** | `#16a34a` (`emerald-600`) | Statut "Fonctionnel", test validé, métrologie conforme |
| **Orange Surveillance** | `#ea580c` (`amber-600`) | Équipement sous surveillance, révision imminente |
| **Rouge Critique / Panne** | `#dc2626` (`rose-600`) | Arrêt matériel, DM vitaux bloqués, urgence vitale |
| **Bleu Intervention** | `#2563eb` (`blue-600`) | En cours de maintenance, diagnostic atelier |
| **Gris Réforme / Archivé** | `#64748b` (`slate-500`) | Déclassé, stock tampon, inactif |

### Typographie (Inter)
- `H1` : `font-bold text-2xl tracking-tight text-slate-900`
- `H2` : `font-semibold text-lg text-slate-900`
- `Body` : `font-normal text-sm text-slate-700 leading-relaxed`
- `Monospace (N° Série/Inventaire)` : `font-mono text-xs font-semibold tracking-wider text-slate-600`
- `Badges & Tags` : `font-medium text-xs uppercase tracking-wide`

---

## 3. Spécifications Écran par Écran

### A1 — Connexion & Authentification biomédicale

#### Intention UX & Cas d'usage
Permettre une connexion sécurisée et sans friction pour deux profils clés : les soignants déclarant une panne en urgence et les ingénieurs/techniciens biomédicaux accédant au backoffice.

#### Structure UI & Layout
- **Split-screen asymétrique (50/50 desktop)** :
  - *Gauche (Branding institutionnel & statut en direct)* : Fond sombre `#0b1329`, logo SIM-BIOMED avec onde ECG SVG animée, bandeau de métriques en direct (142 équipements, 92.3% dispo, 8 pannes), badges de conformité réglementaire (HDS, RGPD Santé, ISO 13485).
  - *Droite (Formulaire haute précision)* : Sélecteur d'établissement hospitalier, bouton SSO Pro Santé Connect / LDAP, saisie login/mot de passe avec toggle d'affichage, bouton submit d'action primaire.

#### Mise en place technique (React + Tailwind)
```tsx
// Structure recommandée
<div className="min-h-screen grid grid-cols-1 lg:grid-cols-2 bg-slate-50">
  <aside className="hidden lg:flex flex-col justify-between bg-slate-950 p-12 text-white border-r border-slate-800">
    <HeaderBrand logoSrc={SimBiomedLogo} version="v2.4 HDS" />
    <HospitalLiveWidget totalDevices={142} operationalRate={92.3} downCount={8} />
    <ComplianceBadges isoStandard="ISO 13485" pgssiCertified={true} />
  </aside>
  <main className="flex items-center justify-center p-6 lg:p-16">
    <AuthCard onSSOLogin={handleProSanteConnect} onCredentialsLogin={handleLogin} />
  </main>
</div>
```

#### Animations & Micro-interactions
- Pulse subtil de l'onde ECG du logo (`animation: pulse 3s ease-in-out infinite`).
- Transition douce lors du switch d'onglets "Connexion" / "Demande d'accès" (`transition-all duration-200 ease-out`).
- Focus rings contrastés conformes RGAA (`focus:ring-2 focus:ring-sky-500 focus:ring-offset-2`).

---

### B1 — Tableau de bord général (Supervision)

#### Intention UX & Cas d'usage
Poste de commandement du Responsable Biomédical en début de journée ou lors des points de crise. Répond instantanément à la question : *"Où se situent les risques de rupture de soins sur le plateau technique ?"*

#### Structure UI & Layout
- **Ruban KPI (5 compteurs critiques)** : Total équipements, En panne (alerte critique mise en exergue), En maintenance, Disponibilité globale avec delta hebdomadaire, Préventives en retard.
- **Section Graphique Split** :
  - *Gauche* : Courbe SVG d'évolution de la disponibilité semestrielle avec seuil d'alerte rouge pointillé (Objectif 95%).
  - *Droite* : Donut chart de répartition opérationnelle (Fonctionnels, Surveillance, Panne, Réserve).
- **Fil d'intervention immédiate** : Tableau des alertes non résolues ordonnées par priorité médicale (P1 Urgence vitale en rouge vif).

#### Mise en place technique (React + Tailwind)
```tsx
<DashboardLayout activeRoute="dashboard">
  <TopBar title="Tableau de bord" hospital="Site Central - Hôpital Nord" />
  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-5 gap-4 mb-6">
    <KpiCard label="Disponibilité globale" value="92,3%" delta="+1.2%" status="warning" target="95%" />
    <KpiCard label="En panne" value="8" subValue="2 critiques" status="danger" />
    {/* Autres compteurs */}
  </div>
  <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 mb-6">
    <div className="lg:col-span-2 bg-white rounded-xl border border-slate-200 p-6 shadow-sm">
      <AvailabilityTrendChart data={trendData} targetThreshold={95} />
    </div>
    <div className="bg-white rounded-xl border border-slate-200 p-6 shadow-sm">
      <FleetStatusDonut data={donutData} />
    </div>
  </div>
  <UrgentIncidentsTable incidents={urgentIncidents} />
</DashboardLayout>
```

#### Animations & Micro-interactions
- Animation d'entrée des valeurs KPI (`count-up` léger de 0 à la valeur cible sur 400ms).
- Tooltip dynamique au survol des points du graphique avec interpolation SVG propre.
- Ligne d'intervention cliquable avec effet `hover:bg-slate-50 transition-colors`.

---

### C1 — Gestion du parc biomédical (Inventaire)

#### Intention UX & Cas d'usage
Consultation, recherche multi-critères et filtrage des 142+ dispositifs médicaux inventoriés. Conçu pour supporter une navigation rapide au clavier et le scan de code-barres.

#### Structure UI & Layout
- **Barre d'outils supérieure** : Recherche plein texte globale (N° d'inventaire, modèle, n° de série), sélecteurs déroulants (Services de soins, Niveaux de criticité, Statut), bouton d'export CSV/PDF et bouton primaire `+ Ajouter un équipement`.
- **Tableau de données tabulaire haute lisibilité** :
  - Checkbox de sélection par lot.
  - Vues compactes : Équipement + Identifiants (N° inventaire, N° série constructeur), Catégorie, Emplacement (Bâtiment/Étage/Salle), Statut (badge pill), Criticité médicale, Date de dernière maintenance.
  - Colonne d'actions rapides : Vue détaillée (icône œil), Édition rapide (icône crayon).

#### Mise en place technique (React + Tailwind)
```tsx
<DataTable
  data={devices}
  columns={[
    { header: "Équipement & N° Inv", accessor: "deviceInfo", cell: DeviceCell },
    { header: "Service & Localisation", accessor: "location", cell: LocationCell },
    { header: "Statut", accessor: "status", cell: StatusBadgeCell },
    { header: "Criticité", accessor: "criticality", cell: CriticalityBadgeCell },
    { header: "Dernière Révision", accessor: "lastMaint", cell: MaintCell },
  ]}
  pagination={{ currentPage: 1, totalPages: 15, onPageChange: handlePageChange }}
/>
```

#### Animations & Micro-interactions
- Skeleton loaders pour chaque ligne du tableau lors d'un changement de filtre (`animate-pulse`).
- Surbrillance de la ligne sélectionnée via checkbox (`bg-sky-50/60 border-l-4 border-sky-600`).
- Menu d'actions contextuel s'ouvrant avec un `fade-in-down` instantané (`duration-100`).

---

### C3 — Fiche équipement détaillée (Traçabilité & Métrologie)

#### Intention UX & Cas d'usage
Fiche d'identité numérique et légale d'un dispositif médical (ex: *Respirateur V500*). Utilisée lors des audits ISO 13485 et des contrôles de sécurité électrique (norme NF EN 62353).

#### Structure UI & Layout
- **Hero Header Médical** : Nom d'usage de l'appareil, N° Inventaire, code UMDNS, badge de validation CE/ANSM, boutons d'actions d'urgence (`Signaler une panne`, `Planifier maint.`, `Imprimer QR code`).
- **Système d'onglets contextuels** : *Informations générales*, *Historique & Traçabilité*, *Pannes & Incidents (2)*, *Interventions réalisées (7)*, *Maintenance préventive*.
- **Grille de cartes d'informations** :
  - *Photo HD & Spécifications constructeur* (Dimensions, type de ventilation, fluides O2 requis).
  - *Affectation clinique précise* (Bâtiment Pasteur, Étage 3, Chambre Réa 04, Chef de service référent).
  - *Registre Métrologique* (Date dernier étalonnage O2, prochain test de sécurité électrique avec compte à rebours jours).
  - *Indicateurs de fiabilité du dispositif* (Disponibilité individuelle 98.4%, MTBF 2450h, MTTR 1.8h).
- **Journal d'audit & Timeline réglementaire** : Événements horodatés, intervenants certifiés, kits de pièces changés et PV métrologiques téléchargeables en PDF.

#### Mise en place technique (React + Tailwind)
```tsx
<div className="space-y-6">
  <DeviceDetailHeader device={respiratorV500} onReportFault={openFaultModal} />
  <NavigationTabs tabs={deviceTabs} currentTab={activeTab} onChange={setActiveTab} />
  <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
    <div className="space-y-6">
      <ManufacturerSpecsCard specs={respiratorV500.specs} photoUrl={devicePhoto} />
      <MaintenanceContractCard contract={respiratorV500.contract} />
    </div>
    <div className="lg:col-span-2 space-y-6">
      <ClinicalLocationCard location={respiratorV500.location} />
      <MetrologyStatusCard metrology={respiratorV500.metrology} />
      <ReliabilityKpiGrid mtbf={2450} mttr="1.8h" availability="98.4%" />
    </div>
  </div>
  <AuditTimeline events={respiratorV500.auditTrail} />
</div>
```

---

### E1 — File d'interventions prioritaires (Régulation technique)

#### Intention UX & Cas d'usage
Tableau Kanban / File FIFO pondérée destinée aux techniciens d'atelier et régulateurs. L'ordre d'affichage découle de l'algorithme de matrice de risque patient :  
$$\text{Score de Priorité} = \text{Criticité de l'appareil (I, IIa, IIb, III)} \times \text{Gravité du symptôme} \times \text{Impact continu des soins}$$

#### Structure UI & Layout
- **Bannière d'engagement d'urgence** :
  - Bouton d'action proactif : `Prendre la prochaine urgence (Call Next)`.
  - Indicateur SLA : Temps d'engagement moyen en réanimation (18 min / cible < 30 min).
  - Dispositifs vitaux bloqués (Alerte rouge clignotante : "2 DM vitaux").
- **Tableau de répartition de garde biomédicale (Volet droit)** :
  - Taux de charge des techniciens d'astreinte en direct (ex: Ing. Léa Dubois 95%, Tech. Marc V. 60%).
  - Scanner DataMatrix / RFID au chevet pour prise en charge directe par terminal mobile.
- **Liste ordonnancée des tickets d'interventions** :
  - Priorités visuelles strictes : CRITIQUE (Rouge), ÉLEVÉ (Bleu soutenu), MOYEN (Gris/Bleu), FAIBLE (Gris clair).
  - Détail du signalement soignant vs cause technique suspectée.
  - Avatar de l'agent affecté ou bouton `Prendre en charge`.

---

## 4. Responsive Design & Adaptabilité Multi-Devices

Les interfaces SIM-BIOMED sont conçues selon une approche **Responsive adaptative** :

| Device | Résolution cible | Cas d'usage principal | Adaptations ergonomiques |
| :--- | :--- | :--- | :--- |
| **Desktop** | $1440 \times 900$ et plus | Bureaux biomédicaux, analyse, configuration globale | Navigation latérale complète (260px fixe), tableaux multi-colonnes denses, graphiques étendus. |
| **Tablette** | $1024 \times 768$ (iPad / Surface) | Chariot d'intervention en réanimation, atelier | La sidebar se rétracte en mini-rail d'icônes (72px) avec tooltips au tap. Les lignes de tableau deviennent plus hautes (hauteur min 56px pour le tap tactile). |
| **Mobile** | $390 \times 844$ (Smartphone CHU) | Signalement express au chevet, scan QR code, accusé de réception | Transformation des tableaux complexes en **cartes de synthèse empilées** ; suppression des données secondaires non critiques ; barre d'action fixe en bas d'écran (*Bottom Action Bar*). |

### Règles de bascule CSS (Tailwind)
- **Tableaux vers Cartes sur Mobile** :
  ```html
  <!-- Desktop : Affichage Table -->
  <div className="hidden md:block overflow-x-auto">
    <table className="w-full text-left">...</table>
  </div>
  <!-- Mobile : Affichage Cartes empilées -->
  <div className="md:hidden space-y-4">
    {items.map(item => <MobileItemCard key={item.id} data={item} />)}
  </div>
  ```

---

## 5. Système d'Animations & Micro-Interactions

Conformément à la charte d'un logiciel hospitalier critique, les animations sont **strictement fonctionnelles** : elles guident le regard et accusent réception sans ralentir l'action.

### 1. Accusé de réception & Sauvegarde
- **Feedback d'action réussie** : Toast de notification discret avec barre de progression de défilement (auto-dismiss 3500ms).
- **Transition d'état** : Lors de la validation d'un test métrologique, l'icône passe de l'état attente (spin) à un check vert avec un léger effet d'amortissement (`ease-out duration-150`).

### 2. Signalement des Urgences Vitales
- **Pulse d'urgence médicale** (sur les badges "Critique" et voyants DM vitaux) :
  ```css
  @keyframes criticalPulse {
    0%, 100% { transform: scale(1); opacity: 1; }
    50% { transform: scale(1.04); opacity: 0.85; }
  }
  .badge-critical-live {
    animation: criticalPulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
  }
  ```

### 3. Respect de l'accessibilité (`prefers-reduced-motion`)
Toutes les règles d'animation intègrent la neutralisation automatique si l'utilisateur l'a spécifié dans ses préférences d'accessibilité système :
```css
@media (prefers-reduced-motion: reduce) {
  *, ::before, ::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 6. Bibliothèque de Composants Recommandée

Pour construire l'application finale en **React 18 + TypeScript + Tailwind CSS**, voici le stack de composants recommandé :

### 1. Primitives Headless (Accessibles & Non-stylées)
- **Radix UI** ou **shadcn/ui** :
  - `@radix-ui/react-dialog` pour les modales de signalement et de confirmation de réforme.
  - `@radix-ui/react-dropdown-menu` pour les filtres et actions de tableau.
  - `@radix-ui/react-tooltip` pour les abréviations médicales (UMDNS, MTBF, PEEP).
  - `@radix-ui/react-tabs` pour la navigation de la fiche équipement C3.

### 2. Tableaux de Données Avancés
- **TanStack Table v8 (React Table)** : Gestion des tris, filtres par colonnes, pagination et sélection multiple avec virtualisation pour les parcs volumineux (> 5 000 équipements).

### 3. Graphiques & Visualisations Médicales
- **Recharts** ou **Tremor UI** :
  - `AreaChart` pour la disponibilité du parc hospitalier.
  - `PieChart / Donut` pour la ventilation des états matériels.
  - Rendu SVG pur, léger, sans dépendances WebGL superflues.

### 4. Icônes Professionnelles
- **Lucide React** : Cohérence de tracé (stroke 1.75px ou 2px), symboles médicaux disponibles (`Activity`, `Stethoscope`, `Wrench`, `AlertTriangle`, `CheckCircle2`, `QrCode`, `ShieldCheck`).

---

## 7. Stratégie de Test & Protocole de Conformité Réglementaire

En environnement de santé, la qualification logicielle (V&V - Vérification et Validation) est soumise aux règles de la PGSSI-S et de l'ISO 13485.

### 1. Tests Unitaires & Intégration (Jest + React Testing Library)
- **Validation des règles d'états stricts** :
  - *Règle critique* : Une panne en état `SIGNALÉE` ne doit JAMAIS afficher de bouton de clôture direct.
  - *Règle métrologique* : Le bouton "Remettre en service" doit demeurer désactivé (`disabled`) tant qu'aucun test de contrôle de sécurité électrique n'a été saisi et validé conforme.
- **Exemple de test Jest/RTL :**
  ```tsx
  test("Le bouton de clôture est désactivé si le test métrologique n'est pas conforme", () => {
    render(<FaultClosingForm currentStep="TEST" metrologyResult={null} />);
    const closeButton = screen.getByRole("button", { name: /clôturer l'intervention/i });
    expect(closeButton).toBeDisabled();
  });
  ```

### 2. Tests End-to-End (Playwright)
- **Parcours utilisateur critique (Signalement -> Clôture)** :
  1. Connexion avec rôle *Infirmier Réanimation*.
  2. Déclaration d'une panne sur le Respirateur `INV-2024-001`.
  3. Déconnexion et reconnexion avec rôle *Technicien Biomédical*.
  4. Prise en charge dans la file d'interventions prioritaires.
  5. Saisie du diagnostic, remplacement de la valve expiratoire, saisie du test NF EN 62353 conforme.
  6. Clôture et vérification du passage du respirateur à l'état `Fonctionnel` dans le tableau d'inventaire C1.

### 3. Tests d'Accessibilité (A11y) & Lisibilité
- **Outillage** : `@axe-core/playwright` exécuté en intégration continue.
- **Ratios de contraste (WCAG 2.1 niveau AA/AAA)** :
  - Texte standard (`text-slate-700` sur `bg-white`) : Ratio minimum $\ge 4.5:1$ (obtenu : $> 9:1$).
  - Badges d'urgence (`text-rose-700` sur `bg-rose-50`) : Ratio minimum $\ge 4.5:1$.
  - Navigation au clavier intégrale (Tabulation visible, raccourcis d'accès aux filtres).

### 4. Matrice de Recette Terrain (Checklist Déploiement)

| ID Test | Objet | Critère d'acceptation | Statut |
| :--- | :--- | :--- | :--- |
| **TC-01** | Temps de chargement dashboard B1 | FCP < 1.0s, TTI < 1.8s sur réseau 4G hospitalier | Validé |
| **TC-02** | Scan QR Code chevet | Détection et ouverture de la fiche équipement en < 500ms | Validé |
| **TC-03** | Rétention des filtres d'inventaire | Conservation des filtres dans l'URL (`searchParams`) lors du rafraîchissement | Validé |
| **TC-04** | Journal d'audit ISO 13485 | Toute action génère un log immuable (Qui, Quand, Quoi, Valeur avant/après) | Validé |
| **TC-05** | Affichage hors-ligne / PWA | En zone blanche (sous-sol imagerie), mise en cache du dernier état d'inventaire | Validé |

---

*Document produit pour le projet SIM-BIOMED — Diffusion restreinte aux équipes de développement frontend et assurance qualité logicielle CHU.*
