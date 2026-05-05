# Changelog

All notable changes to this skill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-05-04

### Added
- Initial public release of the CreativeAI Dataviz Skill.
- Three-step workflow: ingest article URL, propose 5 chart concepts in French, produce a self-contained HTML embed.
- Support for five chart types: bar, line, radar, doughnut, bubble.
- Brand style guide as a JSON token file in `assets/style-guide.json`.
- Canonical HTML embed template documented in `references/html-template.md`.
- Always-on methodology disclaimer in the "À retenir" note when chart data is editorial rather than literal.
- Full editorial wrapper on every output: kicker, title, lede, canvas, note.
- Scoped CSS under unique container IDs to allow multiple charts per page.
- `DOMContentLoaded` retry pattern so embeds work even if Chart.js loads late from the WordPress footer.
- MIT License.
- Four example HTML embeds in `examples/` produced from real CreativeAI.fr articles.

### Notes
The skill ships tuned for the CreativeAI.fr editorial voice. To adapt it to another brand, replace `assets/style-guide.json` with your tokens and adjust the kicker/note conventions in `SKILL.md`. See `docs/customization.md`.
