# How It Works

This document explains the internal architecture of the skill: what each file does, how the workflow runs end to end, and the design decisions behind each rule.

## The three-file architecture

A Claude skill is, at its core, a folder containing one mandatory file (`SKILL.md`) plus any supporting assets and references the workflow needs. This skill ships with three files in two folders.

```
creativeai-dataviz/
├── SKILL.md                       The workflow contract
├── assets/
│   └── style-guide.json           Brand tokens (the source of truth)
└── references/
    └── html-template.md           Canonical embed structure plus per-chart-type Chart.js configs
```

Each file plays a distinct role.

### SKILL.md: the contract

`SKILL.md` is what Claude reads when the skill triggers. Its YAML frontmatter contains the description that determines *when* the skill activates. Its body defines *what* the skill does.

The body is structured into five sections.

1. **Workflow** — the three-step process: ingest the article, propose 5 concepts, produce one HTML embed. Each step is defined as a contract: what the input is, what the output looks like, what the formatting rules are.
2. **Style guide reference** — points Claude to `assets/style-guide.json` and lists the most-used tokens so Claude doesn't need to re-read the full JSON for every chart.
3. **Editorial wrapper** — the five-element structure every embed must include: kicker, title, paragraph, canvas, note.
4. **Methodology disclaimer** — the always-on rule that distinguishes literal data from editorial reading in the "À retenir" note.
5. **What the skill is NOT for** — explicit boundaries that prevent scope creep (no other domains, no static infographics, no fabricated data, no English by default).

### assets/style-guide.json: the source of truth

This file contains every visual token the skill applies: colors, typography, layout dimensions, Chart.js options, dataset configurations, element defaults, mobile breakpoints. The skill never invents a color, a font size, or a border-radius. It pulls them from this JSON.

This discipline is the heart of the system. Replacing `style-guide.json` with another brand's tokens turns the skill into that brand's skill, with no other modifications needed.

The JSON is structured into seven top-level keys, all under `creativeai_chart_style`:

| Key | Purpose |
|---|---|
| `brand_palette` | All brand colors, named semantically |
| `layout` | Container dimensions, canvas heights per chart type, mobile responsive values |
| `typography` | Kicker, title, paragraph, and note styles |
| `chartjs_common_options` | Shared Chart.js options (legend, tooltip) applied to every chart |
| `chartjs_scales` | Cartesian and radar scale configurations |
| `datasets` | Pre-configured dataset styles (primary red, secondary black, multi-palette, etc.) |
| `elements` | Per-chart-type element defaults (bar borderRadius, line tension, doughnut cutout, etc.) |

### references/html-template.md: the executable reference

When Claude produces an HTML embed, it consults this file for the exact wrapper structure and the per-chart-type Chart.js configuration. The file documents:

- The full HTML template with placeholders
- Container width and height tables per chart type
- Five complete Chart.js options blocks (one per supported chart type)
- A pre-delivery checklist

The reference exists because Claude's general knowledge of Chart.js doesn't include CreativeAI.fr's specific calibrations. Without it, Claude would produce technically valid charts that look subtly off-brand. With it, every output is consistent.

## The workflow, end to end

A typical session runs through five logical phases.

### Phase 1: Trigger

The skill activates when the user mentions a CreativeAI.fr URL or asks for "dataviz", "chart ideas", "infographics", or related terms. The activation logic lives in the `description` field of `SKILL.md`'s YAML frontmatter, which Claude evaluates against the user's message.

### Phase 2: Ingest

If a URL is provided, Claude fetches it with `web_fetch`. If the article body is pasted directly, Claude works from that. The skill explicitly handles both inputs.

The reading pass identifies *visualizable structure*: comparison tables, ratings, lists with implicit hierarchy, numeric mentions, taxonomies, before/after pairs, recommendations by use case, scores across models or tools, timelines, prices, performance metrics.

### Phase 3: Propose

Claude returns five distinct chart concepts. Each one names the chart type, the data source inside the article, the editorial angle, and the value to the reader. A short methodology note flags which ideas rely on literal data and which on editorial reading.

The five-concept format is deliberate. One concept would be a guess. Three concepts feel like a poll. Five forces variety: different chart types, different angles, different data shapes. The user can reject all five and the skill loses gracefully (no concept is presented as "the right one").

### Phase 4: Validate (when editorial)

If the user picks a concept that relies on editorial reading, Claude states its data choices explicitly before producing the HTML: what scale, what values, why. This lets the user adjust before the embed gets built. For literal-data concepts, this step is skipped.

### Phase 5: Produce

Claude reads `references/html-template.md` and `assets/style-guide.json` and produces a single HTML block. The block contains:

- A unique container ID (pattern: `cai-chart-{slug}-{shortid}`)
- Scoped CSS under that ID, including mobile breakpoints
- The full editorial wrapper (kicker, title, lede, canvas, note)
- A `<script>` block with the Chart.js instantiation, wrapped in a `DOMContentLoaded` guard with a 200ms retry in case Chart.js loads after the embed renders

The output is ready to paste into a WordPress HTML block. No further editing is needed.

## Design decisions worth knowing

A few choices in the skill exist for reasons that aren't obvious at first read.

### Why no Chart.js CDN script

CreativeAI.fr loads Chart.js once in the site footer. Including a CDN script in every embed would load the library multiple times on a page with multiple charts, slow page load, and create version conflicts when the footer-loaded version differs from the embed-loaded version.

The trade-off: the embed depends on an external dependency it doesn't bundle. The retry guard (`setTimeout(init, 200)`) handles the case where the embed renders before Chart.js arrives.

### Why scoped CSS under a unique ID

Multiple charts on the same article page would collide on every CSS rule if classes were used naively. Scoping every rule under the container's unique ID guarantees that nothing leaks. The pattern is verbose but bulletproof.

### Why French only

The skill is tuned for the CreativeAI.fr editorial voice. The kicker conventions, the "À retenir" framing, the rhythm of the lede paragraph are all French-specific. An English version would need different conventions and would dilute the system's coherence. A future English skill would be a separate skill, not a flag.

### Why the methodology disclaimer is always on (when editorial)

Without it, readers can't tell what's stated in the article and what's the author's interpretation. With it, the chart stays trustworthy even when the data is partly inferred. This is how the system protects the publication's credibility while staying productive.

### Why five chart types, not fifteen

Bar, line, radar, doughnut, and bubble cover the five fundamental data shapes: comparison, trend, profile, distribution, two-axis positioning. Adding more types (treemap, sunburst, sankey, parallel coordinates) would let the skill handle more exotic data but would also dilute its consistency. Five is the constraint.

## When the skill triggers

The skill description, in the YAML frontmatter of `SKILL.md`, is what Claude evaluates to decide whether to activate. The current trigger logic activates the skill when:

- A CreativeAI.fr URL is present in the conversation
- The user mentions "dataviz", "graphique", "chart", "infographic", or "visualisation"
- The user references a creativeai.fr article casually and asks for visual content
- The user wants to add charts, comparisons, or visual data to a creativeai.fr post

It does *not* activate for general dataviz requests on other content. This is intentional. Adapting the skill to another publication means changing the description in addition to the style guide.

## What's deliberately not in scope

A few things the skill won't do, even if asked:

- Generate dataviz for sites other than creativeai.fr
- Create static images or PNG/SVG infographics
- Fabricate numbers the article doesn't contain
- Output English content unless explicitly requested
- Use chart types beyond the five supported (bar, line, radar, doughnut, bubble)
- Bundle Chart.js in the embed

If you need any of those, fork the skill and modify the relevant section of `SKILL.md`.

## Further reading

- [`docs/customization.md`](customization.md) — How to fork this skill for another brand
- [Anthropic Claude Skills documentation](https://docs.claude.com)
- [Chart.js documentation](https://www.chartjs.org/docs/latest/)
