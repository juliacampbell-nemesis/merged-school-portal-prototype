# Merged School Portal — Wireframe Prototype

A clickable wireframe prototype for the merged TimelyCare + Alongside school portal (K-12).

**Live preview**: https://juliacampbell-nemesis.github.io/merged-school-portal-prototype/

## What this is

An exploratory wireframe prototype for the merger of TimelyCare's K-12 portal and Alongside's school portal. Built as a single `index.html` using:

- Tailwind CSS (CDN) configured with Helix Design System tokens
- Adelle Sans (TimelyCare's primary typeface via Adobe Fonts)
- Lucide icons
- Full Helix token system: 8 primitive palettes (tc-navy, tc-sky, tc-berry, tc-orange, tc-sage, tc-gray, tc-cream, tc-light-sand) + semantic tokens (primary, accent, muted, destructive, etc.)

## Navigation

Sidebar (school portal):
- **Home** — daily dashboard with chat summaries, alerts, today's visits, insights
- **Inbox** — Alerts (automated risk detection) + Chat summaries (student-initiated) with detail panels
- **Students** — roster + cohorts (smart and manual groupings)
- **Schools** — district-level school management
- **Library** — Skills · Screeners · Staff resources
- **Reports** — TimelyCampus-style: launcher → individual reports → My Dashboard with pinned widgets
- **Student demo** (collapsible) — preview of the student app
- **Switch to Self-Care** (top of sidebar) — opens the Self-Care app in a new tab

Account dropdown (bottom of sidebar):
- My profile · Settings · Help · Keyboard shortcuts · Switch district · Log out

## Key flows to walk

1. **Inbox triage** — Home → click chat summary → Student profile → Wellness tab
2. **Cohort management** — Students → Cohorts tab → Severe wellness cohort → bulk action toolbar
3. **Report pinning** — Reports launcher → individual report → pin widget icons → My Dashboard
4. **User management (district)** — Account dropdown → Settings → Organization → Users & permissions
5. **Self-Care (separate app)** — top of sidebar → "Switch to Self-Care" (opens new tab)

## Status

Wireframe stage. Helix design system compliance verified against [ux.internal.timelycare.com/helix](https://ux.internal.timelycare.com/helix):
- Color tokens (canonical Higher Ed palette)
- Typography (Adelle Sans, full size scale, H1–H4 hierarchy)
- Badge component (16 variants, primitive color scales)
- Table component (uniform hover, kebab actions, removed redundant View columns)

## Companion artifacts

- **FigJam IA board** — strategic IA discussion with 7+ sections (nav variants, role swimlanes, cohort sub-IA, etc.)
- **`merged-portal-ia.svg`** — early IA diagram for importing into Figma/FigJam
