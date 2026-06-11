---
name: build-playbook-chapter
description: Convert a playbook markdown chapter into a styled HTML page under playbook-html/, with bespoke SVG/CSS visualizations, faithful to the source prose. Use when the user asks to build, generate, or render a playbook chapter as HTML (e.g. "build 02-context-engineering.html", "do the next chapter", "render the Lifecycle chapters").
---

# Build a Playbook Chapter (Markdown → HTML)

Turn one chapter's markdown source into a polished, illustrated HTML page that links the
shared stylesheet and slots into the playbook site. The reference implementation already
exists — match it.

## Canonical references (read these first, every time)

- **Source content** — the chapter's markdown, e.g. `00-foundations/01-why-ai-native.md`.
  The HTML must say what the markdown says. **Never invent claims, numbers, or examples.**
- **Worked example** — `playbook-html/00-foundations/01-why-ai-native.html`. Copy its
  structure, tag order, and class usage. New chapters should feel like siblings of this one.
- **Stylesheet** — `playbook-html/styles.css`. All styling lives here. Read the `CHAPTER PAGES`
  section to see the available classes before writing markup.
- **Index** — `playbook-html/index.html`. Cards already link to every chapter's `.html`; you
  normally don't touch it (see step 6).

## Hard rules

1. **No inline styles, no `<style>` block.** Link the stylesheet only. If a visualization needs
   a style that doesn't exist yet, add a *reusable* class to `styles.css` (in the `CHAPTER PAGES`
   region) — never a one-off inline rule. This is the project's shared-CSS convention.
2. **Output path mirrors the markdown path**, with `.html`: `10-lifecycle/02-plan-before-code.md`
   → `playbook-html/10-lifecycle/02-plan-before-code.html`.
3. **Stylesheet link is depth-relative.** Chapter pages live one folder deep, so use
   `<link rel="stylesheet" href="../styles.css">`. Root pages (`glossary.html`) use `styles.css`.
4. **Every page is self-contained and link-complete** — breadcrumb up, prev/next across, related
   pills out. No dead `href`s except the deliberate disabled prev/next at the ends.
5. **Content fidelity over flourish.** Visualizations illustrate the prose; they never assert
   anything the source doesn't.

## Page skeleton

Use this exact scaffold (see the worked example for a filled-in version):

```html

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{Chapter Title} — The AI-Native Harness Playbook</title>
<link rel="stylesheet" href="../styles.css">
</head>
<body>
<div class="chap">

  <nav class="crumb">
    <a href="../index.html">Playbook</a><span class="sep">›</span>
    <a href="../index.html#{section-id}">{Section Name}</a><span class="sep">›</span>
    <span class="here">{Chapter Title}</span>
  </nav>

  <header class="chap-head">
    <div class="eyebrow">{NN} · {Section} — Chapter {MM}</div>
    <h1>{Chapter Title}</h1>
    <p class="lede">{The chapter's opening blockquote, lightly trimmed. Wrap the punchy
      phrase in <strong> for emphasis.}</p>
  </header>

  <article class="prose">
    <!-- intro paragraphs, then one <h2><span class="idx">NN</span>Heading</h2> per markdown
         "## " section, with figures interleaved (see below) -->
  </article>

  <nav class="chap-nav">
    <a class="prev" href="{prev}.html"><div class="dir">← Previous</div><div class="nm">{Prev Title}</div></a>
    <a class="next" href="{next}.html"><div class="dir">Next →</div><div class="nm">{Next Title}</div></a>
  </nav>

  <div class="related">
    <div class="r-label">Related</div>
    <div class="r-pills"><!-- one <a> per item on the markdown "Related:" line --></div>
  </div>

  <footer>
    <span class="k">Humans steer, agents execute — the harness is the whole game.</span>
    <span><a href="../index.html">← All chapters</a></span>
  </footer>

</div>
</body>
</html>
```

Section ids on the index: `#foundations`, `#lifecycle`, `#harness`, `#delivery`,
`#anti-patterns`, `#adoption`, `#reference`.

At the **first** chapter of a section, give `prev` `class="prev disabled"` and no `href`
(point its `.nm` at the section name). At the **last**, do the same for `next`. Derive
prev/next/related targets from the markdown's footer nav + `Related:` line, converting each
`.md` to `.html` and keeping the relative path (`../10-lifecycle/01-...html`).

## Prose conversion

- Each markdown `## Heading` becomes `<h2><span class="idx">NN</span>Heading</h2>`, numbering
  `01`, `02`, `03`… down the page.
- `**bold**` → `<strong>`, `*italic*` → `<em>`, `` `code` `` → `<code>`. Preserve em dashes.
- A blockquote that's a thesis line becomes a `<p class="pull">` pull quote (wrap the key word
  in `<em>` for the clay accent). Use 1–2 per chapter, max — for the lines that deserve weight.
- Keep paragraphs as written; don't paraphrase. Trim only the markdown's breadcrumb/"Related"
  plumbing, which is rebuilt as real navigation.

## Visualizations — craft 2–4 per chapter

Pick the figure type that fits the content. Wrap each in `<div class="fig"> … <p class="fig-cap">
caption</p></div>`. Available building blocks (all defined in `styles.css`):

- **`.compare` / `.lane.bad` + `.lane.good`** — two-column "wrong way vs right way". `.lane.bad`
  gets a clay bar + `×` bullets; `.lane.good` gets an olive bar + `✓` bullets. Each lane: a
  `.lane-tag` pill, an `<h4>`, a `<ul>` of `<li>`, and a closing `.outcome-line`. Ideal whenever
  the chapter contrasts two approaches.
- **`.pipe`** — a horizontal step pipeline (`.step.c1`…`.c4`, each with `.dot`, `.s-n`, `.s-t`,
  `.s-d`). Arrows are auto-inserted between steps. Use for any sequence/lifecycle.
- **`.outcomes` / `.outcome`** — a 2×2 grid of payoff cards, each with an `.o-icon` SVG, `<h4>`,
  and `<p>`. Use for "the N things you get / the N parts".
- **`.callout`** — a highlighted aside (clay border, tinted bg) with `.c-label` + `<p>`. Use for
  the chapter's single most important takeaway.
- **`.pull`** — pull quote (above).

Invent new figure types when the content calls for it (a ring diagram, a layered stack, a
timeline). When you do, add the class to `styles.css` so it's reusable — don't inline it.

### SVG drawing palette

Inline SVGs inside `.prose` (icons, diagrams) use these helper classes — the **same vocabulary**
as the index thumbnails, now scoped to `.prose svg` too:

| class | meaning |
|-------|---------|
| `.st` | grey outline, no fill (stroke `--g500`) |
| `.ln` | grey line (round cap) |
| `.lc` | clay line (round cap) — the accent stroke |
| `.da` | dashed (combine with a line class) |
| `.fl` | grey fill `--g300` |
| `.cl` `.ol` `.oa` `.sl` `.wh` | fills: clay / olive / oat(+outline) / slate / paper(+outline) |

Prefer these classes over inline `stroke="#…"`/`fill="#…"` so figures stay on-palette and
themeable. Palette hex if ever needed directly: ivory `#FAF9F5`, paper `#FFFFFF`, slate `#141413`,
clay `#D97757`, clay-d `#B85C3E`, oat `#E3DACC`, olive `#788C5D`, greys `#F0EEE6 #E6E3DA #D1CFC5
#87867F #3D3D3A`. Keep SVGs simple, geometric, and legible at small size (the example uses
`viewBox="0 0 32 32"` icons and `0 0 120 80`-ish diagram boxes).

## Step 6 — index card (only if needed)

`index.html` already has a card for every planned chapter, each carrying a one-line `.desc` and a
thumbnail. If the chapter you're building already has its card, leave the index alone. Only edit
it if a card is missing or its description is now wrong.

## Verify before declaring done

1. Serve and screenshot (file:// is blocked in the browser tool):
   ```
   cd playbook-html && python3 -m http.server 8901 >/dev/null 2>&1 &
   ```
   Navigate to `http://localhost:8901/{path}.html`, take a full-page screenshot, and **look at
   it** — check the figures render on-palette, the compare lanes are colored, nav links read
   correctly. Kill the server after (`lsof -ti:8901 | xargs kill`).
2. Confirm prev/next/related/breadcrumb hrefs resolve to real files (or are the deliberate
   disabled ends).
3. Clean up stray artifacts: the browser tool drops a screenshot in the repo root and a
   `playbook-html/.playwright-mcp/` folder — delete them unless asked to keep them.

## Definition of done

The `.html` exists at the mirrored path, links `../styles.css` with zero inline styles, reads
faithfully from the markdown, carries 2–4 on-palette visualizations, is fully navigable in both
directions, and has been eyeballed in a browser screenshot.
