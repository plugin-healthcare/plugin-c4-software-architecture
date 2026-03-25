# AGENTS.md

Summary of all changes and customizations made to this Quarto book project by AI agents.

---

## Project overview

This is a Quarto book using the **arc42 Dutch architecture documentation template** (v8.2-NL).
It renders to both **HTML** and **Typst PDF**, using the `orange-book` Typst extension
(`@preview/orange-book:0.7.1`) as the PDF template.

The brand font is **Figtree**, defined in `_brand.yml` with `source: google`.

---

## Changes made

### 1. Project structure: arc42 Dutch chapter files

Replaced the default Quarto book stubs with 14 Dutch arc42 chapter files sourced from
`arc42-template-NL.md`:

| File | Arc42 section |
|------|--------------|
| `index.qmd` | Voorwoord (preamble) |
| `01-inleiding-doelen.qmd` | 1. Inleiding en doelstellingen |
| `02-architectuur-kaders.qmd` | 2. Architectuur kaders |
| `03-context-systeem-scope.qmd` | 3. Context en systeemscope |
| `04-oplossing-strategie.qmd` | 4. Oplossing strategie |
| `05-bouwstenen-view.qmd` | 5. Bouwstenen view |
| `06-runtime-view.qmd` | 6. Runtime view |
| `07-deployment-view.qmd` | 7. Deployment view |
| `08-crosscutting-concepten.qmd` | 8. Crosscutting concepten |
| `09-architectuur-beslissingen.qmd` | 9. Architectuurbeslissingen |
| `10-kwaliteit-eisen.qmd` | 10. Kwaliteitseisen |
| `11-risicos-technische-schuld.qmd` | 11. Risico's en technische schuld |
| `12-woordenlijst.qmd` | 12. Woordenlijst |
| `references.qmd` | Referenties |

Old stubs (`intro.qmd`, `summary.qmd`) were deleted.

---

### 2. `_quarto.yml` — book metadata and format configuration

- Set `book.title`: `"PLUGIN architectuur"`
- Set `book.author`: `"Daniel Kapitan"`
- Set `book.date`: `"25/03/2026"`
- Updated `book.chapters` to list all 14 arc42 chapter files
- Added `bibliography: plugin.bib`
- Added `brand: _brand.yml`
- **Typst format settings:**
  - `font-paths: fonts` — tells the Typst CLI to look for fonts in the project-local `fonts/`
    directory, ensuring portability without system font installation
  - `mainfont: Figtree` — sets the main font variable passed through pandoc
  - `template-partials: [typst-show.typ]` — instructs Quarto to use the project-local
    `typst-show.typ` instead of the one bundled with the orange-book extension
- **HTML format settings:** `theme: [cosmo, brand]`

---

### 3. `fonts/` directory — embedded Figtree font files

Downloaded all 14 Figtree `.ttf` variants from Google Fonts (via GitHub) into `arc42/fonts/`
so the font is self-contained in the project:

```
fonts/
├── Figtree-Black.ttf
├── Figtree-BlackItalic.ttf
├── Figtree-Bold.ttf
├── Figtree-BoldItalic.ttf
├── Figtree-ExtraBold.ttf
├── Figtree-ExtraBoldItalic.ttf
├── Figtree-Italic.ttf
├── Figtree-Light.ttf
├── Figtree-LightItalic.ttf
├── Figtree-Medium.ttf
├── Figtree-MediumItalic.ttf
├── Figtree-Regular.ttf
├── Figtree-SemiBold.ttf
└── Figtree-SemiBoldItalic.ttf
```

---

### 4. `typst-show.typ` — custom Typst template partial

This is the key customization. The orange-book extension ships its own `typst-show.typ`
(a Pandoc template partial that generates the `#show: book.with(...)` call). Without
overriding it, Figtree cannot be applied to the cover page, TOC, or headings because
orange-book's internal `show heading` and cover page rules execute before the document
body, where any font rules placed by Quarto's brand processing would land.

**Root cause:** In Typst, `set`/`show` rules only apply to content that comes *after*
them in execution order. The orange-book `book()` function renders the cover page and
TOC before processing `body`. Rules injected by Quarto (from brand processing or
`include-in-header`) land inside `body` and therefore cannot reach the cover or TOC.

**Solution:** Create a project-local `typst-show.typ` (referenced via
`template-partials` in `_quarto.yml`) that places font rules *before*
`#show: book.with(...)` so they are in the outer document scope.

The custom `typst-show.typ` adds three rules relative to the orange-book original:

```typst
// BEFORE #show: book.with(...) — outer scope, reaches cover page and TOC
#set text(font: ("Figtree",))
#show outline: set text(font: ("Figtree",))

#show: book.with( ... )

// AFTER #show: book.with(...) — inside body, overrides book()'s inner heading rules
#show heading: set text(font: ("Figtree",))
```

**Why three rules are needed:**

| Rule | Placement | Affects |
|------|-----------|---------|
| `#set text(font: ("Figtree",))` | Before `book.with()` | Cover page title/author/subtitle, all body text |
| `#show outline: set text(font: ("Figtree",))` | Before `book.with()` | Table of contents (rendered inside `book()` before body) |
| `#show heading: set text(font: ("Figtree",))` | After `book.with()` | Headings (orange-book's inner `show heading` rule sets only `size`, not `font`; this re-applies Figtree explicitly in body scope) |

**What did NOT work (and why):**

- `include-in-header` in `_quarto.yml`: injects content before `#show: book.with()` in
  the generated `.typ` file, but at that point `brand-color` and other variables are not
  yet defined, and Quarto's brand processing injects its own `#set text()` and
  `#show heading` rules *after* the header — overriding the header content.
- `_extensions/orange-book/typst-show.typ` without `_extension.yml`: silently ignored
  by Quarto; a local extension directory must have `_extension.yml` to be recognized.
- `#show outline: set text(...)` placed *after* `book.with()` (inside body): the TOC
  is rendered inside `book()` before `body` runs, so body-scoped rules cannot reach it.

---

## File inventory

```
arc42/
├── _quarto.yml              # modified: title, author, chapters, typst font config
├── _brand.yml               # defines Figtree font (source: google); unchanged
├── typst-show.typ           # NEW: custom Pandoc template partial for Typst output
├── fonts/                   # NEW: 14 Figtree .ttf files for self-contained font embedding
├── index.qmd                # modified: Dutch arc42 preamble (Voorwoord)
├── 01-inleiding-doelen.qmd  # NEW
├── 02-architectuur-kaders.qmd # NEW
├── 03-context-systeem-scope.qmd # NEW
├── 04-oplossing-strategie.qmd # NEW
├── 05-bouwstenen-view.qmd   # NEW
├── 06-runtime-view.qmd      # NEW
├── 07-deployment-view.qmd   # NEW
├── 08-crosscutting-concepten.qmd # NEW
├── 09-architectuur-beslissingen.qmd # NEW
├── 10-kwaliteit-eisen.qmd   # NEW
├── 11-risicos-technische-schuld.qmd # NEW
├── 12-woordenlijst.qmd      # NEW
├── references.qmd           # modified: Dutch heading
├── plugin.bib               # bibliography (pre-existing)
└── arc42-template-NL.md     # source reference (unchanged)
```

The `_extensions/orange-book/` directory was created during investigation but is not
used — the override is done via `template-partials` in `_quarto.yml` instead.
