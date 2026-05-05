---
name: creativeai-dataviz
description: Generate Chart.js dataviz embeddable in WordPress HTML blocks for the creativeai.fr blog. Use this skill whenever the user provides a creativeai.fr URL (or pastes article content from that site) and asks for chart ideas, dataviz, infographics, or visual content to enrich an article. Also trigger when the user mentions adding charts, graphs, comparisons, or visual data to a creativeai.fr post, even casually. The skill produces 5 distinct dataviz concepts in French based on the article content, then on user selection produces a ready-to-paste HTML snippet using the CreativeAI brand style guide. Chart.js is already loaded in the site footer, so the snippet only contains the markup, styles, and chart config.
---

# CreativeAI Dataviz Skill

Generates Chart.js dataviz for creativeai.fr articles, embeddable as a WordPress HTML block. Output is in French, follows the CreativeAI style guide, and ships as a self-contained snippet (Chart.js is loaded in the site footer, so no CDN include).

## Workflow

The flow is two-step: **propose 5 concepts**, then **produce one HTML snippet** once the user picks.

### Step 1: Read the article

If the user provides a URL, fetch it with `web_fetch`. If they paste content directly, work from that.

Read the article carefully and extract anything visualizable: comparison tables, ratings, lists with implicit hierarchy, numeric mentions, taxonomies, before/after pairs, recommendations by use case, scores across models or tools, timelines, prices, performance metrics.

The creativeai.fr blog is dense with this kind of structured data. Most articles have at least one explicit comparison table and several implicit ones woven through the prose.

### Step 2: Propose 5 dataviz concepts

Present 5 distinct ideas. Each concept must be:

- **Reader-value first**: the chart should make a point the prose alone doesn't make as fast. SEO is a side effect of that, not the goal.
- **Grounded in the article**: only use data actually present in the source. Never fabricate numbers or ratings.
- **Distinct from the others**: don't propose 5 bar charts. Vary the angle (comparison vs distribution vs hierarchy vs trade-off vs recommendation).
- **Picked for fit, not novelty**: choose the chart type that best matches the data shape. The brand style guide supports bar, line, radar, doughnut, and bubble. Pick what fits.

Format for each concept (in French):

```
**Idée N — [Titre court et clair]**
Type : [bar / line / radar / doughnut / bubble]
Donnée source : [d'où vient la data dans l'article]
Ce que ça raconte : [1 phrase, l'angle éditorial]
Pourquoi c'est utile au lecteur : [1 phrase]
```

After the 5 concepts, add a short methodology note that flags which ideas rely on **literal data** from the article vs **editorial reading** of the article. This sets expectations honestly and lets the user know they may need to validate numbers before publication.

End with: "Lequel je code en HTML ?"

### Step 3: Produce the chosen HTML

Once the user picks, output a single HTML block ready to paste in a WordPress HTML block. Read `assets/style-guide.json` for the exact style tokens and apply them strictly. See `references/html-template.md` for the full structural template.

If the chosen idea relies on editorial reading rather than literal numbers, **state your data choices explicitly** before producing the HTML — what scale you used, what values you assigned, why. Give the user a chance to adjust before delivery. Do this in a short paragraph, not a table or a full explanation: 3–5 lines is enough.

Hard rules for the output:

- Do **not** include a `<script src="...chart.js...">` tag. Chart.js is already loaded in the site footer.
- Do include all the CSS inline in a `<style>` block scoped to a unique container ID, so the snippet is self-contained and won't collide with other charts on the page.
- Use a unique `canvas` ID per chart (e.g. `chart-pellicules-radar-001`) to allow several charts on one page.
- Wait for `DOMContentLoaded` before instantiating the chart, since the snippet may render before Chart.js loads.
- All text in French: kicker, title, paragraph, axis labels, legend, tooltip, note.
- Mobile breakpoint and responsive heights from the style guide are mandatory.
- Numbers and ratings must come from the article. If the article uses star ratings (⭐⭐⭐⭐), convert them to numeric (4/5 etc.) and say so in the note.

## Style guide

`assets/style-guide.json` is the source of truth for colors, typography, layout, Chart.js options, scales, datasets, elements, and mobile breakpoints. Read it before producing the HTML snippet. Do not invent colors, fonts, or border-radii — pull them from the JSON.

Quick reminders:

- Brand red `#cc1f1f` is the accent. Black `#111111` and grey `#666666` are the secondary tones. Use the multi-palette only when you genuinely need 4+ series.
- Bar chart `borderRadius: 8`, line `tension: 0.42`, doughnut `cutout: 60%`, radar scale `0–10`.
- Container max-width 760px (or 860px for radar/bubble), border-radius 14px, soft shadow.
- The kicker is a small uppercase pill at the top, brand-red text on light grey.
- The note block has a 4px black left border and lives below the chart.

## Editorial wrapper

Every snippet ships with the full wrapper:

1. **Kicker** — short uppercase tag, broad category name, not the article's SEO keyword (e.g. "PELLICULES ARGENTIQUES", "GEM CRITIQUE CRÉATIF", "CUSTOM GPTS VS GEMS GEMINI"). The kicker frames the topic, not the chart.
2. **Title** — the chart's editorial headline, not its technical description.
3. **Paragraph** — 1–2 sentences setting up what the chart shows.
4. **Canvas** — the chart itself.
5. **Note** — "À retenir" block with the key takeaway. The takeaway should be actionable for the reader, not a description of the chart.

## Methodology disclaimer (always present by default)

When the chart's data is **editorial** (your reading of the article rather than literal numbers stated in the text), the note **must** include a brief methodology line so the reader knows what they're looking at. This is the default behavior — do not skip it.

Examples of acceptable phrasings:

- "Estimation éditoriale basée sur l'importance accordée à chaque pilier dans l'article — pas une mesure scientifique."
- "Notation 0 = absent, 1 = partiel, 2 = présent, basée sur les fonctionnalités décrites dans l'article."
- "Notes converties d'étoiles ⭐ en valeurs 0–5 pour permettre la comparaison."

When the chart's data is **literal** (numbers, ratings, or counts stated explicitly in the article), the disclaimer is not required, but a short source pointer is welcome ("Source : tableau de référence de l'article").

The disclaimer goes at the end of the note, after the takeaway. Format: takeaway first (1 sentence), then the methodology line, separated by a period and a space. The "À retenir :" prefix opens the note in bold.

## What this skill is NOT for

- Generating dataviz for sites other than creativeai.fr (the style guide is brand-specific)
- Creating images or static infographics (this is Chart.js only)
- Inventing data the article doesn't contain
- Producing English content unless the user explicitly asks
