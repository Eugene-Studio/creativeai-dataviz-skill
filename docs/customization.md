# Customizing the skill for another brand

This skill is built around CreativeAI.fr, but the architecture is portable. If you publish on WordPress, have a brand style guide, and want a Claude skill that turns your articles into branded dataviz, this document walks you through the fork.

The work is genuinely small. Most of the effort goes into the style guide JSON, which you probably already have in some form (a Figma design tokens file, a Notion brand page, or a CSS variables sheet).

Plan on **two to four hours** for the first pass, plus iteration as you test on real articles.

## Step 1: Fork the repo

Fork `creativeai-dataviz-skill` on GitHub, or clone it locally and rename it. The directory you'll be modifying is `creativeai-dataviz/`.

Rename the skill folder to match your brand:

```
your-brand-dataviz/
├── SKILL.md
├── assets/
│   └── style-guide.json
└── references/
    └── html-template.md
```

The folder name becomes the skill's identifier in Claude's UI. Use a short, lowercase, hyphen-separated name.

## Step 2: Replace the style guide

`assets/style-guide.json` is the file you'll spend the most time on. Open the existing CreativeAI.fr file, keep its **structure** intact, and replace every value with your brand equivalents.

The structure must stay the same because `SKILL.md` and `references/html-template.md` reference its keys explicitly. If you rename `brand_palette.red_primary` to `brand_palette.accent_blue`, the references break.

What to replace:

- `brand_palette` — your full color list, semantically named
- `layout` — container max-widths, canvas heights per chart type, padding, border-radius, box-shadow, font-family
- `typography` — kicker, title, paragraph, and note styles
- `chartjs_common_options` — legend position and styling, tooltip background and border colors
- `chartjs_scales` — axis tick colors, gridline colors, font sizes
- `datasets` — your equivalents to the CreativeAI.fr `primary_red`, `secondary_black`, `multi_palette` configurations
- `elements` — bar `borderRadius`, line `tension`, doughnut `cutout`, etc. Adjust to match your visual identity.
- `mobile` — your mobile breakpoint and responsive heights

Tip: keep your accent color in a single semantic key (e.g. `brand_palette.accent_primary`) and reference it consistently in datasets. If you later change the accent, you only update one value.

## Step 3: Update SKILL.md

Open `creativeai-dataviz/SKILL.md` and edit four specific sections.

### 3a. The frontmatter description

The YAML `description` at the top of the file determines when the skill activates. Replace mentions of "creativeai.fr" with your domain, and adjust the trigger phrases to match how your contributors will request charts.

For example, if your publication is named "VisualLab" and lives at `visuallab.io`:

```yaml
description: Generate Chart.js dataviz embeddable in WordPress HTML blocks for the VisualLab blog. Use this skill whenever the user provides a visuallab.io URL...
```

The description is the activation contract. Make it specific.

### 3b. The language

The current skill outputs French. If your publication is in another language, update every reference in `SKILL.md` to point to that language. Specifically:

- The "All text in French" rule in the hard rules
- The example phrases in the editorial wrapper section ("PELLICULES ARGENTIQUES", "À retenir :")
- The methodology disclaimer examples

Replace the French phrasings with your language's equivalents. Keep the structure the same.

### 3c. The kicker conventions

The kicker is the small uppercase pill at the top of every chart. The CreativeAI.fr convention is *broad category names, not SEO keywords* (e.g. "PELLICULES ARGENTIQUES" rather than "GUIDE PORTRA 400"). Decide what your brand's kicker should look like, document it in `SKILL.md`, and provide three or four examples drawn from your real articles.

### 3d. The "À retenir" note

The note at the bottom of every embed has a fixed structure: takeaway first, methodology line second when the data is editorial. The label "À retenir :" is brand-specific. Replace it with your equivalent (English: "Key insight" or "TL;DR" or "Takeaway"; Spanish: "Para recordar"; etc.).

If your brand has its own signature label for editorial callouts on its website, use the same one here. Consistency between the chart embeds and the rest of the publication's voice matters.

## Step 4: Update html-template.md

`references/html-template.md` documents the HTML wrapper and the Chart.js options. Two updates are needed.

### 4a. The container ID prefix

The pattern is currently `cai-chart-{slug}-{shortid}`. The `cai-` prefix is CreativeAI.fr's namespace. Replace it with your brand's two-to-four-letter prefix (e.g. `vl-chart-` for VisualLab) so embeds across multiple skills don't collide if a page ever runs charts from different sources.

### 4b. The container width and height table

Verify the per-chart-type dimensions still match your `style-guide.json`. If you changed `canvas_wrap.height_radar` from 560px to 600px in the JSON, update the table here too. The two files must agree or charts will render at the wrong size.

## Step 5: Repackage the skill

Once your three files are updated, repackage the skill into a `.skill` file. The skill-creator scripts handle this:

```bash
# Clone the skill-creator repo if you haven't
git clone https://github.com/anthropics/skill-creator.git
cd skill-creator

# Package your skill
python -m scripts.package_skill /path/to/your-brand-dataviz/
```

The script validates the skill structure, then produces `your-brand-dataviz.skill` in the current directory. Drag that file into Claude (Settings → Capabilities → Skills) to install.

## Step 6: Test on real articles

Run the skill on at least three of your own articles before considering it production-ready. Watch for:

- **Kicker mismatches** — does the kicker convention you defined feel right when applied to actual articles?
- **Color distribution** — do your dataset palettes look right when applied to real data, or does one color dominate awkwardly?
- **Mobile rendering** — open the embed on a 375px-wide viewport. Are headings legible? Does the canvas height feel right?
- **Methodology disclaimer** — does the "À retenir" note read naturally with your editorial voice, or does it feel grafted on?

Iterate on the JSON and the SKILL.md based on what you find. The skill is small enough that changes ship in seconds.

## Step 7: Publish your skill

Two options.

**Public fork.** Push your fork to GitHub with a permissive license (MIT recommended), update the README with your brand's credentials, and let other publications use it as their starting point. This builds authority for your brand in the open-source space.

**Private skill.** Keep the repo private and use the `.skill` file internally. This makes sense if your style guide is confidential, or if the skill encodes editorial choices you'd rather not publish.

Either path is valid. The skill works the same way.

## Common adaptations

A few common cases worth flagging.

### English-language publications

Beyond changing the language, English requires shorter sentences in the lede and the note (English packs less density per character than French). Plan for 1–3 sentence ledes and 1–2 sentence notes.

### Multi-language publications

Don't try to make one skill bilingual. Build two skills (one per language), each with its own SKILL.md. The activation triggers will distinguish them based on the URL or the user's request language.

### Publications with multiple brand sub-identities

If your publication has sub-brands with different visual identities (e.g. a main brand and a research division with a different color palette), build one skill per sub-brand. Each skill points to its own `style-guide.json`. The skill descriptions disambiguate based on URL paths or category mentions.

### Publications without a strict style guide

If your publication doesn't have a formalized style guide yet, this is a good reason to build one. The skill forces the question: what *are* your brand colors? What's your default chart container width? Answering these questions for the skill will improve your visual consistency across the rest of the site too.

## Need help?

If you'd rather have the skill built for you, [Eugène Studio](https://www.eugene-studio.com/lab/dataviz-skill) designs editorial systems on commission. Typical timeline: two to three weeks. Deliverable: a packaged skill, a private repo, a documentation handoff.

For technical questions about Claude skills in general, the [Anthropic documentation](https://docs.claude.com) is the authoritative source.
