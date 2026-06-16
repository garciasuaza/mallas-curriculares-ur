# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

Static HTML visualization platform for the **curricular reform of 4 undergraduate programs** (ANI, ECON, FIN, MND) at Facultad de Economía, Universidad del Rosario. No build system, no dependencies — all files are self-contained HTML with inline CSS/JS, deployed to GitHub Pages.

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
| `nbc` | `web/nbc.html` | Núcleo Básico Común detail |
| `ped` | `web/pedagogia.html` | Pedagogical principles |
| `act` | `web/actividades.html` | Reform activities tracker |
| `men` | `web/menores.html` | Undergraduate minors |
| `ice` | `web/icesi_mallas_comparativo.html` | Icesi reference |

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

**Programs**: `ani` (140 cr), `econ` (128 cr), `fin` (126 cr), `mnd` (140 cr), `enb` (128 cr — PROPUESTA).

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

Two active minors, each a `.minor-tab` + `.minor-pane` + `<table class="courses-table">`:
- **Menor en Experience Marketing** — 6 courses, 12 cr, all 2 cr (ADM · MND)
- **Menor en Economía de la Empresa** — 5 courses, 12 cr (ANI · MND · FIN · ECON) — badge PROPUESTA `#7C3AED`

MND's malla shows these as `Profundización / Minor` blocks (not individual courses); the detail lives only in menores.html.

Tab switching: `showMinor(key)` toggles `.active` on tabs and panes by id `mtab-{key}` / `minor-{key}`.

## Brand Guidelines

- **Red**: `#DA0921` · **Navy**: `#190056` · **Purple**: `#3100A0` · **Orange**: `#E8670C`
- Typography: Calibri (body), Bebas Neue (display headings, via Google Fonts)
- ENB program uses `#7C3AED` (violet) to signal PROPUESTA status

## articulacion.html — Data Structure

`SHARED` array lists every course that appears in 2+ programs. Each entry:
```js
{ blq:'nbc', n:'Course name', cr:3, ANI:4, ECON:null, FIN:2, MND:4, note:'optional divergence note' }
```
`blq` is the canonical block for that course. When updating after an Excel change, also update `bloques.html` (`DATA` object) if any program distribution changed.

## bloques.html — Updating Credit Totals

`DATA` object has one entry per program with credits per block. Must match the `bloques` field in `mallas_programas.html` exactly:
```js
FIN: { ros:18, nbc:33, dcom:12, desp:30, elec:21, grad:12, prac:0, total:126 }
```

## Source Data

Curricula come from `Propuesta_ANI_Econ_Fin_MND jun15.xlsx` (parent folder — **current version**). ENB from `Propuesta_ENB_EconomiaDeLosNegocios_v01.xlsx` + `gen_enb.py`. Minors from `Menores Pregrado.xlsx` (sheets EC04, FI01, ADM1).

When a new Excel version arrives, use openpyxl to read cell fill colors:
- `FFF4B183` = ros · `FF9DC3E6` = nbc · `FFFFD966` = dcom · `FFA9D08E` = desp · `FFB4A7D6` = elec · `FFFFF2CC` = grad · `FFE2EFDA` = prac

Update `mallas_programas.html` first, then `bloques.html` (totals), then `articulacion.html` (shared courses).

**Current program credit totals (jun15):**
| Program | ros | nbc | dcom | desp | elec | grad | prac | total |
|---------|-----|-----|------|------|------|------|------|-------|
| ANI | 18 | 30 | 17 | 48 | 12 | 9 | 6 | 140 |
| ECON | 18 | 21 | 18 | 35 | 24 | 12 | 0 | 128 |
| FIN | 18 | 33 | 12 | 30 | 21 | 12 | 0 | 126 |
| MND | 18 | 30 | 14 | 51 | 12 | 9 | 6 | 140 |

## Deployment

```bash
git add web/<file>.html
git commit -m "description"
git push  # triggers GitHub Pages rebuild (~1 min)
```
