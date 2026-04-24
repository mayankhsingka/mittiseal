# MittiSeal — Codebase Cleanup Plan

Status: **pending** · created 2026-04-24 · owner: Claude + manku
Pick this up when we have budget for ~60–90K tokens in one session.

---

## Goal

Keep the site as plain HTML, but extract shared design tokens and components
into reusable files so adding new pages (video, graphics, team, gallery, etc.)
doesn't mean duplicating 1,000 lines of CSS each time.

**Non-goals.** No framework migration. No build step. No JS bundler. No
renaming of visible UI or restructuring the hero/scenes layout.

---

## Current state (2026-04-24)

```
website/
├── index.html            2,354 lines — home page, all CSS inline (~1,000 lines)
├── MittiSeal.pdf         source proposal document
├── annexure/
│   ├── index.html        annexures listing page
│   ├── style.css         partial shared stylesheet (duplicates some tokens)
│   └── 1..9/index.html   nine annexure detail pages
└── FUTUREPLAN.md         this file
```

Pain points:
- `:root` design tokens (`--green`, `--gold`, `--clay`, `--paper`, etc.) are declared in both `index.html` and `annexure/style.css`. Any palette change has to be made in two places.
- Base typography (`.display`, `.h1`, `.eyebrow`, `.caption`, `.lede`) is duplicated.
- No `assets/` folder yet — future video and SVG graphics have nowhere clean to live.
- Scroll/reveal animation CSS, card patterns, and table styles exist in both files with minor drift.

---

## Target state

```
website/
├── index.html              home page — scene CSS still inline, but links to base.css
├── assets/
│   ├── base.css            shared tokens + typography + common components
│   ├── img/                (empty — for future raster images, prefer .webp)
│   ├── video/              (empty — only for self-hosted; prefer YT/Vimeo embed)
│   └── svg/                (empty — for future inline SVG graphics)
├── annexure/
│   ├── style.css           annexure-specific rules only; imports base.css
│   ├── index.html
│   └── 1..9/index.html
├── MittiSeal.pdf
└── FUTUREPLAN.md
```

`assets/base.css` contains everything that's genuinely shared across every page:
1. `:root` custom properties (all design tokens)
2. CSS reset + body base
3. Noise texture overlay (`body::before`)
4. Typography classes: `.display`, `.h1`, `.h2`, `.eyebrow`, `.caption`, `.caption-italic`, `.lede`, `.body`
5. Scene-dark / scene-green colour modifiers
6. Reveal / stagger animations
7. Scroll progress bar
8. Nav-rail (the right-edge dots)

Everything else — hero layout, problem scene, scale scene, etc. — stays
inline in `index.html` since it's home-specific.

---

## Step-by-step plan

### Step 1 — Read and map (input-heavy)
- Read `index.html` lines 15–~1500 (the `<style>` block).
- Make a two-column mental list: **shared** (moves to base.css) vs **home-specific** (stays).
- No file writes in this step.

### Step 2 — Create `assets/base.css`
- Write the extracted shared block, preserving exact selectors and values.
- Match the formatting style of the existing inline CSS for low diff noise.

### Step 3 — Slim `index.html`
- Remove the blocks that migrated to `base.css`.
- Add `<link rel="stylesheet" href="assets/base.css">` in `<head>` before the remaining `<style>` block.
- Do **not** touch the HTML body, scripts, or hero layout.

### Step 4 — Rewrite `annexure/style.css`
- Remove the `:root` block, reset, and typography classes (now in base.css).
- Keep only annexure-specific rules: `.wrap`, `.topbar`, `.grid`, `.card`, `.stats`, `.table-wrap`, `.steps`, `.callout`, `.annex-foot`.
- Update every annexure HTML file so `<head>` links both stylesheets, in order:
  ```html
  <link rel="stylesheet" href="../../assets/base.css">
  <link rel="stylesheet" href="../style.css">
  ```
  (annexure/index.html uses `../assets/base.css` and `style.css`)

### Step 5 — Create empty asset folders
- `assets/img/`, `assets/video/`, `assets/svg/` — each with a `.gitkeep` so git tracks them.

### Step 6 — Smoke test
- Open `index.html`, `annexure/index.html`, `annexure/1/index.html`, `annexure/9/index.html` in a browser.
- Verify: colour palette identical, typography identical, nav-rail still works, reveal animations still fire, annexure cards still hover correctly.
- If anything drifts visually, the fix is almost certainly a missed selector in `base.css`.

### Step 7 — Commit
- One commit: `refactor: extract shared CSS into assets/base.css`
- No behaviour change. Pure structure.

---

## Files that will change

| File | Change |
|---|---|
| `assets/base.css` | **new** — ~400–500 lines |
| `assets/{img,video,svg}/.gitkeep` | **new** — empty |
| `index.html` | remove ~400 lines of CSS, add 1 `<link>` |
| `annexure/style.css` | remove ~60 lines, add 1 `@import` or rely on HTML-side link |
| `annexure/index.html` | add 1 `<link>` |
| `annexure/1..9/index.html` | add 1 `<link>` each (9 small edits) |

---

## Risks and how to handle them

1. **Visual drift.** A selector gets left behind or moved but overridden. Mitigation: side-by-side before/after comparison of home + one annexure in browser before commit. If drift appears, diff the extracted block against the original inline block — it's almost always a missed rule.

2. **CSS load order matters.** Annexure pages must load `base.css` *before* `annexure/style.css` so scene-specific overrides still win. Enforce via HTML `<link>` order, not `@import` (more reliable).

3. **File-path mistakes.** `/annexure/1/index.html` sits two levels deep, so `../../assets/base.css`. Easy to get wrong. Grep for `assets/base.css` after edits and eyeball every relative path.

4. **Token budget overrun.** If Step 1 alone exceeds 30K tokens, narrow the scope to Light mode (tokens + typography only, ~25–40K total) and defer the rest to another session.

---

## Token budget

Estimated total: **60–90K tokens**.

- Step 1 read: ~25K input
- Step 2 write base.css: ~6K output
- Step 3 index.html edits: ~8K (several Edit calls)
- Step 4 annexure updates: ~10K (9 files + style.css)
- Steps 5–7: minimal

If we have 150K+ free budget at session start, proceed. If less, do the
Light variant: just extract `:root` tokens and base typography, link from
every page, skip component extraction. That version is ~30K tokens and
still eliminates the palette-in-two-places problem.

---

## How to resume this

1. Read this file.
2. Open `index.html` at its current length — if it's still ~2,354 lines with inline `<style>`, the plan is unstarted; proceed from Step 1.
3. If `assets/base.css` already exists, check which step we reached and continue from there.
4. Always verify the visible site looks identical after the refactor. The whole point is zero UX change.
