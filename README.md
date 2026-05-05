# creativeai-dataviz-skill

A Claude skill that turns CreativeAI.fr articles into Chart.js dataviz, embeddable as WordPress HTML blocks.

Drop a URL, get five distinct chart concepts in French. Pick one, get a self-contained HTML embed that respects the brand style guide and ships ready to paste.

![License: MIT](https://img.shields.io/badge/license-MIT-black) ![Built with Claude Skills](https://img.shields.io/badge/Claude-Skill-cc1f1f) ![Made by Eugène Studio](https://img.shields.io/badge/by-Eug%C3%A8ne%20Studio-111111)

---

## What it does

The skill runs in three steps inside Claude.

1. **Ingest.** You paste a CreativeAI.fr URL. The skill fetches the article and reads it for visualizable structure: comparison tables, ratings, thresholds, taxonomies, recommendations.
2. **Propose.** It returns five distinct chart concepts in French. Each concept names the chart type, the source data inside the article, the editorial angle, and the value to the reader. A short methodology note flags which ideas rely on literal data versus editorial reading.
3. **Produce.** You pick one. The skill returns a single HTML block ready to paste in a WordPress HTML block: scoped CSS, unique IDs, the full editorial wrapper (kicker, title, lede, canvas, "À retenir" note), and a `DOMContentLoaded` guard. Chart.js is expected to be loaded in the site footer, so no CDN script is included.

## Why this exists

CreativeAI.fr publishes long-form guides on generative AI, creative direction, and AI workflows. Most articles contain implicit comparisons that the prose alone cannot surface fast: response benchmarks, model preferences, frameworks with weighted components, decision matrices.

A reader scans an article in 90 seconds. A chart works in three. Without a system, charts get made one by one, slowly, with inconsistent style. With this skill, the same workflow runs in under five minutes per article, and every output respects the brand style guide.

## Live examples

Four charts produced by this skill, embedded in real CreativeAI.fr articles.

| Article | Chart | Type |
|---|---|---|
| Custom GPTs : le guide complet | Les 5 piliers d'un Custom GPT face aux Gems | Bar grouped |
| Gems Gemini : le guide complet | La grille de notation sur 10 du Critique Créatif | Bar horizontal |
| Anamorphoses Z-Axis avec Nano Banana Pro | La fenêtre de lisibilité des mots Z-Axis | Line |
| Tree of Thought : 30 prompts | ToT vs CoT, préférences humaines | Doughnut |

Open `examples/` in this repo to see the standalone HTML files.

## Install

You need Claude with skills enabled (claude.ai or Claude Desktop).

1. Download `creativeai-dataviz.skill` from this repo.
2. In Claude, open Settings → Capabilities → Skills.
3. Drag the `.skill` file into the upload area.
4. Open a new conversation. Paste a CreativeAI.fr URL and ask for dataviz ideas. The skill triggers automatically.

That is the entire installation. The skill is self-contained.

## How it works

The skill folder ships three files.

```
creativeai-dataviz/
├── SKILL.md                       The workflow, hard rules, methodology disclaimer guidance
├── assets/
│   └── style-guide.json           Brand tokens: colors, typography, layout, Chart.js options
└── references/
    └── html-template.md           Canonical HTML wrapper plus per-chart-type Chart.js configs
```

`SKILL.md` defines the three-step workflow and the rules that govern every output: French only, full editorial wrapper, no Chart.js CDN, unique container IDs for multi-chart pages, methodology disclaimer always present when data is editorial rather than literal.

`assets/style-guide.json` is the source of truth for visual design. The skill never invents colors or typography; it pulls them from this file. To adapt the skill to another brand, this is the file you replace.

`references/html-template.md` documents the canonical embed structure plus the Chart.js options block for bar, line, radar, doughnut, and bubble charts. Each option block is calibrated against the brand tokens.

For more, see [docs/how-it-works.md](docs/how-it-works.md).

## Customize for another brand

The skill is built around CreativeAI.fr but the architecture is portable. To adapt it to your own publication:

1. Fork the repo.
2. Replace `creativeai-dataviz/assets/style-guide.json` with your brand tokens (same JSON shape).
3. Edit the description and the per-section examples in `SKILL.md` to match your editorial voice.
4. Repackage the skill (see [docs/customization.md](docs/customization.md)).

If you want a custom skill built for your brand, [Eugène Studio](https://www.eugene-studio.com/lab/dataviz-skill) designs editorial systems on commission.

## Known limits

The skill assumes a CreativeAI.fr-shaped article: long-form, dense, in French. It will work on other sources but the kicker conventions and the "À retenir" framing are tuned for this voice.

The five concepts always include a literal-vs-editorial breakdown. Trust the literal ones immediately. Validate the editorial ones (numeric estimates, weighting choices) before publication. The skill makes its assumptions visible; it does not pretend to certainty it does not have.

Chart.js must be loaded elsewhere on the page. The skill's HTML output does not bundle it. On WordPress, the cleanest setup is to enqueue Chart.js in the theme footer.

## License

MIT. Use it, fork it, ship your own version. If you build something with it, a link back is appreciated but not required.

## Built by Eugène Studio

Eugène Studio designs creative systems at the intersection of generative AI and visual storytelling. This skill is part of a broader case study published at [eugene-studio.com/lab/dataviz-skill](https://www.eugene-studio.com/lab/dataviz-skill).

Author: Christophe Martin · [christophe-martin.xyz](https://www.christophe-martin.xyz) · [LinkedIn](https://www.linkedin.com/in/christophemartin)
