# HTML Template for CreativeAI Dataviz

This is the canonical structure for a single embeddable Chart.js block on creativeai.fr. The snippet is self-contained: scoped CSS, unique IDs, and a `DOMContentLoaded` guard. Chart.js is loaded in the site footer, so do **not** include a CDN script tag.

## Naming conventions

- Container ID pattern: `cai-chart-{slug}-{shortid}` (e.g. `cai-chart-pellicules-radar-a1b2`)
- Canvas ID pattern: same as container with `-canvas` suffix
- Use the same slug as a CSS class scope so styles don't leak

## Full template

```html
<div class="cai-chart cai-chart--{TYPE}" id="cai-chart-{SLUG}-{SHORTID}">
  <style>
    #cai-chart-{SLUG}-{SHORTID} {
      max-width: 760px;
      margin: 2.5rem auto;
      padding: 1.5rem 1.25rem;
      border: 1px solid #e5e5e5;
      border-radius: 14px;
      background: #ffffff;
      box-shadow: 0 12px 30px rgba(0, 0, 0, 0.08);
      font-family: -apple-system, BlinkMacSystemFont, "Inter", "Helvetica Neue", Arial, sans-serif;
      color: #111111;
    }
    #cai-chart-{SLUG}-{SHORTID} .cai-header {
      margin-bottom: 1.4rem;
    }
    #cai-chart-{SLUG}-{SHORTID} .cai-kicker {
      display: inline-flex;
      align-items: center;
      margin-bottom: 0.65rem;
      padding: 0.28rem 0.6rem;
      border-radius: 999px;
      background: #f7f7f7;
      color: #cc1f1f;
      font-size: 0.76rem;
      font-weight: 700;
      letter-spacing: 0.04em;
      text-transform: uppercase;
    }
    #cai-chart-{SLUG}-{SHORTID} .cai-title {
      margin: 0 0 0.5rem 0;
      font-size: 1.45rem;
      line-height: 1.25;
      color: #111111;
    }
    #cai-chart-{SLUG}-{SHORTID} .cai-lede {
      margin: 0;
      font-size: 0.96rem;
      line-height: 1.6;
      color: #555555;
    }
    #cai-chart-{SLUG}-{SHORTID} .cai-canvas-wrap {
      position: relative;
      width: 100%;
      height: {HEIGHT}; /* see height table below */
      margin: 0 auto;
    }
    #cai-chart-{SLUG}-{SHORTID} canvas {
      display: block;
      width: 100% !important;
      height: 100% !important;
    }
    #cai-chart-{SLUG}-{SHORTID} .cai-note {
      margin-top: 1rem;
      padding: 0.85rem 1rem;
      border-left: 4px solid #111111;
      border-radius: 6px;
      background: #f7f7f7;
      color: #333333;
      font-size: 0.9rem;
      line-height: 1.55;
    }
    @media (max-width: 600px) {
      #cai-chart-{SLUG}-{SHORTID} {
        padding: 1.1rem 0.9rem;
        border-radius: 12px;
      }
      #cai-chart-{SLUG}-{SHORTID} .cai-title {
        font-size: 1.22rem;
      }
      #cai-chart-{SLUG}-{SHORTID} .cai-canvas-wrap {
        height: {MOBILE_HEIGHT};
      }
    }
  </style>

  <div class="cai-header">
    <span class="cai-kicker">{KICKER_TEXT}</span>
    <h3 class="cai-title">{TITLE_TEXT}</h3>
    <p class="cai-lede">{LEDE_TEXT}</p>
  </div>

  <div class="cai-canvas-wrap">
    <canvas id="cai-chart-{SLUG}-{SHORTID}-canvas"></canvas>
  </div>

  <div class="cai-note"><strong>À retenir :</strong> {NOTE_TEXT}</div>

  <script>
  (function () {
    function init() {
      var ctx = document.getElementById('cai-chart-{SLUG}-{SHORTID}-canvas');
      if (!ctx || typeof Chart === 'undefined') {
        // Chart.js not yet loaded — retry once
        return setTimeout(init, 200);
      }
      new Chart(ctx, {
        type: '{CHART_TYPE}',
        data: {
          /* labels + datasets here */
        },
        options: {
          /* options here, see per-type configs below */
        }
      });
    }
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', init);
    } else {
      init();
    }
  })();
  </script>
</div>
```

## Container width by chart type

| Chart type | max-width | desktop height | mobile height |
|---|---|---|---|
| bar | 760px | 340px | 320px |
| line | 760px | 380px | 360px |
| radar | 860px | 560px | 500px |
| bubble | 860px | 500px | 440px |
| doughnut | 760px | 430px | 460px |

Radar and bubble use the larger 860px container. The rest use 760px.

## Per-type Chart.js options

These are the canonical `options` blocks. Copy and adjust labels/colors only.

### Bar

```js
options: {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { display: false },
    tooltip: {
      backgroundColor: '#111111', titleColor: '#ffffff', bodyColor: '#ffffff',
      borderColor: '#cc1f1f', borderWidth: 1, padding: 10, cornerRadius: 6
    }
  },
  scales: {
    x: { grid: { display: false }, ticks: { color: '#111111', font: { size: 13, weight: '500' } } },
    y: { beginAtZero: true, grid: { color: 'rgba(0,0,0,0.08)' }, ticks: { color: '#666666', font: { size: 11 } } }
  }
}
```

Bar dataset defaults: `borderRadius: 8`, `borderSkipped: false`, `borderWidth: 1`. Use `bar_palette` from the style guide JSON for multi-color bars, or solid `#cc1f1f` for a single-series chart.

### Line

```js
options: {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { position: 'bottom', labels: { boxWidth: 12, boxHeight: 12, padding: 14, color: '#111111', font: { size: 12 } } },
    tooltip: { backgroundColor: '#111111', titleColor: '#ffffff', bodyColor: '#ffffff', borderColor: '#cc1f1f', borderWidth: 1, padding: 10, cornerRadius: 6 }
  },
  scales: {
    x: { grid: { display: false }, ticks: { color: '#111111', font: { size: 13, weight: '500' } } },
    y: { beginAtZero: true, grid: { color: 'rgba(0,0,0,0.08)' }, ticks: { color: '#666666', font: { size: 11 } } }
  },
  elements: { line: { tension: 0.42 } }
}
```

Line dataset: `tension: 0.42`, `fill: true`, `borderWidth: 3`, `pointRadius: 5`, `pointHoverRadius: 7`. Primary line uses `primary_red` dataset config from style guide.

### Radar

```js
options: {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { position: 'bottom', labels: { boxWidth: 12, boxHeight: 12, padding: 14, color: '#111111', font: { size: 12 } } },
    tooltip: { backgroundColor: '#111111', titleColor: '#ffffff', bodyColor: '#ffffff', borderColor: '#cc1f1f', borderWidth: 1, padding: 10, cornerRadius: 6 }
  },
  scales: {
    r: {
      beginAtZero: true, min: 0, max: 10,
      ticks: { display: true, stepSize: 2, color: '#777777', backdropColor: 'rgba(255,255,255,0.85)', font: { size: 10 } },
      grid: { color: 'rgba(0,0,0,0.10)' },
      angleLines: { color: 'rgba(0,0,0,0.12)' },
      pointLabels: { color: '#111111', font: { size: 13, weight: '500' } }
    }
  },
  elements: { line: { tension: 0.05 } }
}
```

Radar scale is fixed 0–10. If the source data is on a different scale (e.g. 0–5 stars), normalize to 0–10 and mention the conversion in the note.

### Doughnut

```js
options: {
  responsive: true,
  maintainAspectRatio: false,
  cutout: '60%',
  plugins: {
    legend: { position: 'bottom', labels: { boxWidth: 12, boxHeight: 12, padding: 14, color: '#111111', font: { size: 12 } } },
    tooltip: { backgroundColor: '#111111', titleColor: '#ffffff', bodyColor: '#ffffff', borderColor: '#cc1f1f', borderWidth: 1, padding: 10, cornerRadius: 6 }
  }
}
```

Doughnut dataset: `borderWidth: 3`, `borderColor: '#ffffff'`, `hoverOffset: 10`. Use `multi_palette` for slice colors.

### Bubble

```js
options: {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { position: 'bottom', labels: { boxWidth: 12, boxHeight: 12, padding: 14, color: '#111111', font: { size: 12 } } },
    tooltip: { backgroundColor: '#111111', titleColor: '#ffffff', bodyColor: '#ffffff', borderColor: '#cc1f1f', borderWidth: 1, padding: 10, cornerRadius: 6 }
  },
  scales: {
    x: { grid: { color: 'rgba(0,0,0,0.08)' }, ticks: { color: '#666666', font: { size: 11 } }, title: { display: true, color: '#333333', font: { size: 13 } } },
    y: { grid: { color: 'rgba(0,0,0,0.08)' }, ticks: { color: '#666666', font: { size: 11 } }, title: { display: true, color: '#333333', font: { size: 13 } } }
  }
}
```

Bubble points use `r` (radius) per data point. Default `r: 6`, hover `r: 8`. Always label both axes via `scales.x.title.text` and `scales.y.title.text`.

## Final checks before delivering

- Every `{PLACEHOLDER}` is replaced with real content
- No `<script src="...chart.js...">` tag (footer-loaded)
- All text is in French
- Container ID and canvas ID are unique
- Source data is traceable to the article
- Mobile heights are set
- The "À retenir" note has substance (not "ce graphique montre…")
