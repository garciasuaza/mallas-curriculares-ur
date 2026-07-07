# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

Static HTML visualization platform for the **curricular reform of 6 undergraduate programs** (ANI, ECON, FIN, MND, ADM, EMP) at Facultad de Economía, Universidad del Rosario. No build system, no dependencies — all files are self-contained HTML with inline CSS/JS, deployed to GitHub Pages.

**Live URL**: `https://garciasuaza.github.io/mallas-curriculares-ur/`

## Local Preview

```bash
python -m http.server 7432
# then open http://localhost:7432
```

The `.claude/launch.json` configures this server for the preview tool.

## Architecture

**`index.html`** is the tab-router shell: sticky header (UR brand) + sticky navbar, switches `display:block/none` on `.pane` divs each containing one `<iframe>`. Tab state synced to `location.hash`. The `TABS` array must list every key in registration order.

**Navbar** uses dropdowns: `toggleDrop(id)` opens/closes a `.dd-menu`. The `DD_PARENT` map routes each tab key to its parent dropdown id so the active dropdown stays open on navigation. Current mapping:
```js
DD_PARENT = {
  mallas:'dd-reforma', art:'dd-reforma', blq:'dd-reforma',
  nbc:'dd-recursos',   ped:'dd-recursos', act:'dd-recursos', men:'dd-recursos',
  ice:'dd-referencia'
}
```
Three dropdown groups: **Reformas** (`dd-reforma`) · **Recursos** (`dd-recursos`) · **Referencia** (`dd-referencia`). `inicio` is a direct link (no dropdown).

The NBC tab label in navbar and `inicio.html` is **"Syllabus nuevos cursos"** (not "Núcleo Básico Común").

Each tab needs: a `.tab-btn` with `id="tab-{key}"`, a `.pane` with `id="pane-{key}"`, the key added to `TABS`, and an entry in `DD_PARENT` if inside a dropdown.

Cross-iframe navigation from child pages uses `parent.switchTab('key')`.

## Current Tabs (in order)

| Key | File | Content |
|-----|------|---------|
| `inicio` | `web/inicio.html` | Dashboard KPI, program cards, timeline |
| `mallas` | `web/mallas_programas.html` | Semester-by-semester curricula |
| `art` | `web/articulacion.html` | Cross-program shared courses |
| `blq` | `web/bloques.html` | Credits by block type |
| `nbc` | `web/nbc.html` | Núcleo Básico Común — 5 courses (selector + left/right panel) |
| `ped` | `web/pedagogia.html` | Pedagogical principles |
| `act` | `web/actividades.html` | Reform activities tracker |
| `men` | `web/menores.html` | Undergraduate minors |
| `ice` | `web/icesi_mallas_comparativo.html` | Icesi reference |

## mallas_programas.html — Nav & Legend

Program nav uses **pill style V3**: `.prog-tab` has `border-radius:20px`, active state is `box-shadow: inset 0 -3px 0 var(--ur-red)` (not a red fill). ENB tab sits at the end with `border-left:2px solid var(--ur-gray-e)` separator.

Per-pane legend: each program pane has a `<div class="prog-legend" id="pleg-{key}">` populated by JS at init from the `BLOQUES` array. No global legend bar.

Tab labels: `ECON — Economía`, `FIN — Finanzas` (with program code prefix). Nav order: ANI · ECON · FIN · MND · ADM · EMP · ENB.

## mallas_programas.html — Data Structure

All curriculum data lives in a single `PROGRAMS` JS object. Each program has:
```js
programKey: {
  sems: [
    { n: 1, total: 18, cursos: [
      { n: 'Course name', cr: 3, blq: 'nbc', tag: 'optional subtitle' },
    ]},
    { n: 8, empty: true },  // empty semester
  ],
  bloques: { ros:18, nbc:30, dcom:21, desp:41, elec:9, grad:9, total:128 }
}
```

**Programs**: `ani` (140 cr), `econ` (128 cr), `fin` (126 cr), `mnd` (140 cr), `enb` (128 cr — PROPUESTA), `adm` (140 cr), `emp` (140 cr).

Program tab dot colors: ANI `#DA0921` · ECON `#3100A0` · FIN `#1D4ED8` · MND `#E8670C` · ENB `#7C3AED` · ADM `#0D9488` · EMP `#B45309`.

Adding a program: add entry to `PROGRAMS`, add tab button to `prog-nav`, add `prog-pane` div, add key to the init `forEach`.

## Block Type Color Convention

The display label for `dcom` is **"Disciplinar-Departamento"** everywhere (mallas, bloques, articulacion).

| Key | Label | Border color | Background |
|-----|-------|-------------|------------|
| `ros` | Formación Rosarista | `#DA0921` | `#FEE8EA` |
| `nbc` | Núcleo Básico Común | `#3100A0` | `#EDE9FE` |
| `dcom` | Disciplinar-Departamento | `#065F46` | `#D1FAE5` |
| `desp` | Disciplinar específica | `#B8460A` | `#FEF0E7` |
| `elec` | Electividad | `#713F12` | `#FEF9C3` |
| `grad` | Opción de grado | `#0C4A6E` | `#E0F2FE` |
| `prac` | Prácticas | `#4D1A7A` | `#F5F0FE` |

## menores.html — Current Minors

Two active minors with tab switching via `showMinor(key)` → ids `mtab-{key}` / `minor-{key}`:

- **Menor en Experience Marketing** (`mkt`) — 6 courses, 12 cr, simple `<table class="courses-table">` (ADM · MND)
- **Menor en Economía de la Empresa** (`eco`) — anchor+pool structure, badge PROPUESTA `#7C3AED`:
  - **Anchor card** per program: fixed course each program brings (FIN has no fixed anchor — picks 4 freely from pool)
  - **Pool table** (`.pool-table`): 11-row availability matrix showing which pool courses each program can take
  - Each program selects anchor (if any) + enough pool courses to reach 5 total / 12 cr

MND's malla shows these as `Profundización / Minor` blocks (not individual courses); the detail lives only in menores.html.

## Brand Guidelines

- **Red**: `#DA0921` · **Navy**: `#190056` · **Purple**: `#3100A0` · **Orange**: `#E8670C`
- Typography: Calibri (body), Bebas Neue (display headings, via Google Fonts)
- ENB program uses `#7C3AED` (violet) to signal PROPUESTA status
- ADM uses `#0D9488` (teal) · EMP uses `#B45309` (amber)

## articulacion.html — Data Structure

`SHARED` array lists every course that appears in 2+ programs. Each entry:
```js
{ blq:'nbc', n:'Course name', cr:3, ANI:4, ECON:null, FIN:2, MND:4, ADM:4, EMP:4, note:'optional divergence note' }
```
The `note` field documents block-classification divergences (e.g. EMP classifies "Data Driven" as dcom while other programs use desp). The matrix in `renderMatrix()` computes all C(6,2)=15 pairs dynamically from `PROGS`.
`blq` is the canonical block for that course. When updating after an Excel change, also update `bloques.html` (`DATA` object) if any program distribution changed.

## bloques.html — Updating Credit Totals

`DATA` object has one entry per program with credits per block. Must match the `bloques` field in `mallas_programas.html` exactly:
```js
FIN: { ros:18, nbc:33, dcom:12, desp:30, elec:21, grad:12, prac:0, total:126 }
```

## Source Data

Curricula come from:
- ANI/ECON/FIN/MND: `Propuesta_ANI_Econ_Fin_MND jun15.xlsx` (parent folder — **current version**)
- ADM/EMP: `Propuesta_mallas_010726.xlsx` (parent folder)
- ENB: `Propuesta_ENB_EconomiaDeLosNegocios_v01.xlsx` + `gen_enb.py`
- Minors: `Menores Pregrado.xlsx` (sheets EC04, FI01, ADM1)

When a new Excel version arrives, use openpyxl to read cell fill colors:
- `FFF4B183` = ros · `FF9DC3E6` = nbc · `FFFFD966` = dcom · `FFA9D08E` = desp · `FFB4A7D6` = elec · `FFFFF2CC` = grad · `FFE2EFDA` = prac

Update `mallas_programas.html` first, then `bloques.html` (totals), then `articulacion.html` (shared courses).

**Current program credit totals:**
| Program | ros | nbc | dcom | desp | elec | grad | prac | total | Source |
|---------|-----|-----|------|------|------|------|------|-------|--------|
| ANI | 18 | 30 | 17 | 48 | 12 | 9 | 6 | 140 | jun15 |
| ECON | 18 | 21 | 18 | 35 | 24 | 12 | 0 | 128 | jun15 |
| FIN | 18 | 33 | 12 | 30 | 21 | 12 | 0 | 126 | jun15 |
| MND | 18 | 30 | 14 | 51 | 12 | 9 | 6 | 140 | jun15 |
| ADM | 18 | 30 | 17 | 48 | 12 | 9 | 6 | 140 | 010726 |
| EMP | 18 | 30 | 18 | 47 | 12 | 9 | 6 | 140 | 010726 |

**Key block divergences vs ANI** (ADM/EMP): Finanzas Corporativas → nbc (not dcom); IA for International Business → dcom (ADM/EMP) vs desp (ANI); EMP: Innovación y Emprendimiento → desp (not dcom); EMP: Data Driven → dcom (not desp).

## nbc.html — Course Selector Pattern

Five courses: `micro`, `macro`, `prob`, `ens`, `ia`. Each course has a `.sel-btn` (top bar), a `.course-pane` (left panel — ficha + RAEs + accordion unidades + tags), and a `.pdf-pane` (right panel — iframe or placeholder). The `COURSES` array in JS controls switching. Adding a course: add sel-btn, course-pane, pdf-pane, and the key to `COURSES`.

## actividades.html — Structure

Simple card grid, no expand/collapse. Two sections: **Diseño curricular** (8 activities) and **Procesos de soporte** (6 activities). Cards use orange left border (`.act-card`), navy border (`.act-card.support`), or dashed gray (`.act-card.pending-act`) for unassigned. No JS needed — static display only.

## Deployment

```bash
git add web/<file>.html
git commit -m "description"
git push  # triggers GitHub Pages rebuild (~1 min)
```
