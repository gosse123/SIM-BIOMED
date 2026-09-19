---
name: SIM-BIOMED Design System
colors:
  surface: '#f8f9ff'
  surface-dim: '#cbdbf5'
  surface-bright: '#f8f9ff'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#eff4ff'
  surface-container: '#e5eeff'
  surface-container-high: '#dce9ff'
  surface-container-highest: '#d3e4fe'
  on-surface: '#0b1c30'
  on-surface-variant: '#3f4850'
  inverse-surface: '#213145'
  inverse-on-surface: '#eaf1ff'
  outline: '#707881'
  outline-variant: '#bfc7d2'
  surface-tint: '#006398'
  primary: '#006194'
  on-primary: '#ffffff'
  primary-container: '#007bb9'
  on-primary-container: '#fdfcff'
  inverse-primary: '#93ccff'
  secondary: '#565e74'
  on-secondary: '#ffffff'
  secondary-container: '#dae2fd'
  on-secondary-container: '#5c647a'
  tertiary: '#006577'
  on-tertiary: '#ffffff'
  tertiary-container: '#008096'
  on-tertiary-container: '#f9fdff'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#cce5ff'
  primary-fixed-dim: '#93ccff'
  on-primary-fixed: '#001d31'
  on-primary-fixed-variant: '#004b73'
  secondary-fixed: '#dae2fd'
  secondary-fixed-dim: '#bec6e0'
  on-secondary-fixed: '#131b2e'
  on-secondary-fixed-variant: '#3f465c'
  tertiary-fixed: '#acedff'
  tertiary-fixed-dim: '#4cd7f6'
  on-tertiary-fixed: '#001f26'
  on-tertiary-fixed-variant: '#004e5c'
  background: '#f8f9ff'
  on-background: '#0b1c30'
  surface-variant: '#d3e4fe'
typography:
  headline-xl:
    fontFamily: Inter
    fontSize: 2rem
    fontWeight: '700'
    lineHeight: 2.5rem
    letterSpacing: -0.02em
  headline-xl-mobile:
    fontFamily: Inter
    fontSize: 1.5rem
    fontWeight: '700'
    lineHeight: 2rem
    letterSpacing: -0.015em
  headline-lg:
    fontFamily: Inter
    fontSize: 1.5rem
    fontWeight: '600'
    lineHeight: 2rem
    letterSpacing: -0.01em
  headline-lg-mobile:
    fontFamily: Inter
    fontSize: 1.25rem
    fontWeight: '600'
    lineHeight: 1.75rem
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Inter
    fontSize: 1.25rem
    fontWeight: '600'
    lineHeight: 1.75rem
  headline-sm:
    fontFamily: Inter
    fontSize: 1rem
    fontWeight: '600'
    lineHeight: 1.5rem
  body-lg:
    fontFamily: Inter
    fontSize: 1rem
    fontWeight: '400'
    lineHeight: 1.5rem
  body-md:
    fontFamily: Inter
    fontSize: 0.875rem
    fontWeight: '400'
    lineHeight: 1.375rem
  body-sm:
    fontFamily: Inter
    fontSize: 0.75rem
    fontWeight: '400'
    lineHeight: 1.125rem
  label-lg:
    fontFamily: Inter
    fontSize: 0.875rem
    fontWeight: '600'
    lineHeight: 1.25rem
  label-md:
    fontFamily: Inter
    fontSize: 0.75rem
    fontWeight: '600'
    lineHeight: 1rem
    letterSpacing: 0.02em
  label-sm:
    fontFamily: Inter
    fontSize: 0.6875rem
    fontWeight: '600'
    lineHeight: 0.875rem
    letterSpacing: 0.03em
rounded:
  sm: 0.125rem
  DEFAULT: 0.25rem
  md: 0.375rem
  lg: 0.5rem
  xl: 0.75rem
  full: 9999px
spacing:
  gutter: 1rem
  gutter-desktop: 1.5rem
  margin: 1rem
  margin-desktop: 2rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 0.75rem
  space-lg: 1.25rem
  space-xl: 2rem
---

## Brand & Style

The design system embodies the rigor, precision, and vital urgency of hospital and biomedical engineering. Designed for biomedical engineers, hospital technicians, department heads, and clinical staff, it balances mission-critical utility with modern, cognitive-load-reducing clarity. The aesthetic follows a **Corporate & Clinical Precision** model: razor-sharp hierarchy, elevated dark slate structural frames contrasting against pristine white data surfaces, and unambiguous color coding that communicates asset health and failure severity at a single glance.

Key attributes:
- **Clinical Reliability:** Crisp information grouping, predictable interaction pathways, and zero decorative noise. Every pixel reinforces trust and regulatory traceability.
- **Immediate Diagnostic Clarity:** High-contrast telemetry badges and operational status tags allow technical personnel to triage emergencies within seconds.
- **Ergonomic Density:** Structured layout systems crafted for dense tabular data, multi-parameter monitoring, and touch-compatible mobile interventions on hospital floors.

## Colors

The palette is engineered around high-reliability biomedical workflows. The navigation framework uses deep slate and navy tones (`#0F172A`, `#1E293B`) to anchor the interface and provide visual permanence. Content areas sit on soft clinical backdrops (`#F8FAFC`, `#F1F5F9`) with pure white card surfaces (`#FFFFFF`).

### Functional State Colors
State colors are strictly regulated and never used purely for decoration:
- **Operational / Functional / Up-to-Date (`#10B981`):** Indicates normal equipment operation, successful preventive maintenance, and resolved tickets. Soft background: `#ECFDF5`, border: `#A7F3D0`.
- **Warning / Medium Priority / Surveillance (`#F59E0B`):** Warns of upcoming calibration dates, moderate wear, or secondary priority incidents. Soft background: `#FFFBEB`, border: `#FDE68A`.
- **Critical / Breakdown / Urgent Alert (`#EF4444`):** Flags acute equipment breakdowns, vital sign monitor failures in ICU, or emergency repair needs. Soft background: `#FEF2F2`, border: `#FECACA`.
- **In Progress / Diagnostic / Intervention (`#3B82F6`):** Active on-site or in-lab technician work. Soft background: `#EFF6FF`, border: `#BFDBFE`.
- **Scheduled / Pending (`#8B5CF6`):** Planned interventions and pending parts orders. Soft background: `#F5F3FF`, border: `#DDD6FE`.

### Interactive and Accent Colors
- **Primary Action (`#0284C7` / `#0369A1`):** Focused interactive states, primary action buttons, key indicators, and active table filters.
- **Cyan / Telemetry Accent (`#06B6D4` / `#0EA5E9`):** ECG pulse visualizers, active workflow nodes, telemetry badges, and sensor status beacons.

## Typography

Inter is chosen for its neutral legibility, exceptional numeral clarity (critical for serial numbers, MTBF, and MTTR telemetry), and extensive weight support across compact spaces.

Rules for implementation:
- **Numerical Data & Metrics:** Always enable tabular numbers (`font-variant-numeric: tabular-nums`) across table rows, telemetry counters, serial code displays, and inventory IDs to eliminate optical jumping during real-time updates.
- **Status Badges & Labels:** Use `label-md` or `label-sm` with semibold weighting and slight positive letter spacing (`0.02em` - `0.03em`) to enhance scan-efficiency in high-pressure clinical settings.
- **Section & Panel Headers:** Keep headers strictly functional. Avoid dramatic sizes; prioritize compact vertical density over editorial scale.

## Layout & Spacing

The application architecture utilizes an asymmetrical fixed-sidebar layout coupled with a responsive fluid viewport. 

- **Desktop Structure:**
  - Sidebar: Fixed width of `240px` (`260px` collapsed to `72px` on compact screens), deep navy tone `#0F172A`.
  - Global Header: Fixed height of `64px`, sticky position, housing asset global search, hospital department picker, active incident notifications, and user session badge.
  - Workspace Canvas: 12-column dynamic fluid grid with a standard `1.5rem` gutter and `2rem` outer margin. Maximum canvas width is capped at `1680px` for ultra-wide clinical monitors to preserve line-length legibility.
- **Tablet / Mobile Structure (Floor Technicians):**
  - Breakpoints: Mobile (< 768px), Tablet (768px - 1024px), Desktop (> 1024px).
  - On mobile devices, the left sidebar transforms into a bottom utility bar (quick scan barcode/QR, quick breakdown ticket, asset search, active tasks).
  - Single column reflow for card widgets and intervention tickets with fixed bottom CTA buttons for field reports.

## Elevation & Depth

Visual hierarchy leverages a hybrid of **Tonal Layering** and **Low-Contrast Micro-Borders** tailored for bright, sterile hospital lighting conditions. Heavy blurry drop shadows are prohibited; interfaces rely on subtle border definitions and hairline separators.

- **Level 0 (Canvas Base):** Arrière-plan clinique (`#F8FAFC`). Flat, non-interactive foundation.
- **Level 1 (Card & Module Surfaces):** Pure White (`#FFFFFF`) with a 1px micro-border (`#E2E8F0` or `#CBD5E1`). Optional faint floor shadow: `0 1px 3px 0 rgba(15, 23, 42, 0.04), 0 1px 2px -1px rgba(15, 23, 42, 0.02)`.
- **Level 2 (Hover States & Interactive Panels):** Elevates subtle border contrast to `#94A3B8` with an ambient glow: `0 4px 6px -1px rgba(15, 23, 42, 0.07), 0 2px 4px -2px rgba(15, 23, 42, 0.04)`.
- **Level 3 (Modals, Slide-over Diagnostic Drawers, Popovers):** Pure white body floating with high-precision drop shadow: `0 20px 25px -5px rgba(15, 23, 42, 0.12), 0 8px 10px -6px rgba(15, 23, 42, 0.06)`, bounded by a 1px border `#CBD5E1`.
- **Telemetry Layers (ECG / Pulse Accents):** Dark surfaces (`#0F172A`, `#1E293B`) use inner cyan highlights (`0 0 12px rgba(6, 182, 212, 0.25)`) to denote active electrical and operational heartbeat sensors.

## Shapes

The design system maintains a **Soft / High-Precision** geometry (`roundedness: 1`). Medical instruments and regulatory software necessitate crisp, dependable geometric boundaries rather than casual, ultra-pill curves.

- **Standard Containers & Cards:** `0.375rem` to `0.5rem` (`6px` - `8px`) radius. Offers a balanced, technical feel.
- **Buttons, Inputs & Form Controls:** `0.375rem` (`6px`) radius, creating crisp alignment with neighboring table fields.
- **Telemetry Status Badges:** `9999px` (Fully rounded pills) strictly for status pills and category indicators, distinguishing metadata chips immediately from rectangular buttons and form fields.
- **Inner Interactive Icons & Avatars:** `0.375rem` or full circle for technician profiles.

## Components

### Buttons
- **Primary:** Solid Medical Blue (`#0284C7`), hover (`#0369A1`), active (`#075985`). Text: Pure white, `label-lg` weight, height `38px` (compact) or `44px` (touch-target mobile).
- **Secondary:** Surface `#FFFFFF`, border `1px solid #CBD5E1`, text `#1E293B`. Hover: background `#F1F5F9`, border `#94A3B8`.
- **Critical / Danger (Declare Breakdown):** Solid `#EF4444`, hover `#DC2626`. White label.
- **Ghost / Tertiary:** Transparent background, text `#0284C7`, hover `#F0F9FF`.

### Status Badges & Priority Tags
Composed of a container, a pulsing or solid circular status dot (6px), and a concise label:
- **Functional:** `#ECFDF5` background, `#10B981` border/dot, `#065F46` text.
- **Critical / Breakdown:** `#FEF2F2` background, `#EF4444` border/dot, `#991B1B` text.
- **Monitoring / Attention:** `#FFFBEB` background, `#F59E0B` border/dot, `#92400E` text.
- **In Progress / Repair:** `#EFF6FF` background, `#3B82F6` border/dot, `#1E40AF` text.
- **Scheduled:** `#F5F3FF` background, `#8B5CF6` border/dot, `#5B21B6` text.

### Biomedical Workflow Stepper (Timeline)
Represents the 7-step lifecycle: **Signalement → Qualification → Criticité → Diagnostic → Intervention → Test → Clôture**.
- **Completed Steps:** Filled `#0284C7` circle with check icon, linked by a solid `#0284C7` line (2px).
- **Active Step:** Ringed `#0284C7` with cyan `#06B6D4` pulsing center dot and bold title.
- **Upcoming Steps:** Subtle slate border `#CBD5E1`, white interior, neutral text `#94A3B8`.
- Supports horizontal layout on desktop detail views and vertical collapsible timeline on technician mobile screens.

### Data Tables (Parc & Interventions)
- **Header:** Background `#F8FAFC`, uppercase `label-sm` font, text `#64748B`, height `40px`, bottom border `1px solid #E2E8F0`.
- **Row:** Height `48px` to `56px`, hover background `#F8FAFC`, transition 150ms. Zebra striping is disabled in favor of soft divider lines (`#F1F5F9`).
- **Cells:** Tabular numbers for inventory codes (`INV-2024-001`), aligned equipment thumbnails (`40x40px` with soft border), and nested badge components for quick scanning.

### KPI Metric Cards
- Header displaying metric label (`label-md`, `#64748B`) and contextual icon.
- Main value (`headline-xl`, `#0F172A`, tabular numbers) e.g., `MTBF 124 h` or `Disponibilité 92,3%`.
- Trend sub-badge: pill badge showing delta percentage (e.g., `+1,2%` with `#10B981` text and micro green arrow).

### Form Inputs & Barcode Scanners
- Standard inputs: Border `1px solid #CBD5E1`, background `#FFFFFF`, text `#0F172A`, placeholder `#94A3B8`. Focus ring: `2px solid #0284C7` with `0 0 0 3px rgba(2, 132, 199, 0.15)`.
- Quick-Scan input: Suffix button featuring medical camera/QR scan icon triggering instant mobile camera capture for asset identification.