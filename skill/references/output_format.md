# GARGON Output Format (v2)

## Overview

The GARGON output is an HTML file with a **side-by-side layout**: the original article on the left, the GARGON sidebar on the right. The article text is unchanged. GARGON adds personalized context in the sidebar, aligned section-by-section. Each section is a horizontal row with the article panel and sidebar panel side by side.

## Key v2 Features

1. **Side-by-side layout** — article left (flex: 1), GARGON sidebar right (340px fixed width)
2. **GARGON acronym bar** — spelled-out acronym above the sidebar column
3. **Friction heatmap** — SVG document icon in header showing per-section difficulty as colored stripes
4. **Sticky tone toggle** — fixed-position toggle (Standard / Plain Speak) that follows the user down the page
5. **Collapsible mini-reports** — each section's sidebar starts collapsed; the summary line shows concept count, types, and need-to-read score
6. **Highlighted terms in article text** — `.gargon-term` spans with hover tooltips that swap when tone changes
7. **Dual-tone scaffolds** — every scaffold has a `data-conv` attribute with the Plain Speak version; JS swaps innerHTML on toggle
8. **Entry type labels** — use the updated taxonomy: `term`, `framework`, `notation`, `meaning divergence`, `cross-domain transfer`, `recognition gap`, `analogy`
9. **Responsive** — stacks vertically on screens under 900px

## HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GARGON — [Article Title]</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    font-family: 'Georgia', 'Times New Roman', serif;
    line-height: 1.7;
    color: #1a1a1a;
    background: #fafaf8;
  }

  /* Header — flex row: title left, friction heatmap right */
  .gargon-header {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem 2rem 1.5rem;
    border-bottom: 2px solid #2c5f2d;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }
  .header-left { flex: 1; }
  .gargon-header h1 {
    font-size: 1.1rem; font-weight: 600; color: #2c5f2d;
    letter-spacing: 0.1em; text-transform: uppercase; margin-bottom: 0.5rem;
  }
  .gargon-header .article-title {
    font-size: 1.6rem; font-weight: 700; color: #1a1a1a; margin-bottom: 0.75rem;
  }
  .gargon-header .meta { font-size: 0.85rem; color: #666; line-height: 1.5; }
  .gargon-header .meta span { display: inline-block; margin-right: 1.5rem; }

  /* Side-by-side section rows */
  .section-row {
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    gap: 0;
    border-top: 1px solid #e0e0e0;
  }

  /* Article panel (left) */
  .article-panel {
    flex: 1;
    padding: 1.5rem 2rem;
    min-width: 0;
    order: 1;
  }

  /* GARGON sidebar (right) */
  .gargon-sidebar {
    width: 340px; min-width: 340px;
    padding: 1.25rem 2rem 1.25rem 1.25rem;
    background: #f0f7f0;
    border-left: 3px solid #2c5f2d;
    order: 2;
  }
  .gargon-sidebar.no-context {
    background: #f8f8f5;
    border-left: 3px solid #ccc;
  }

  /* Section label */
  .section-label {
    font-size: 0.7rem; font-weight: 700; color: #2c5f2d;
    text-transform: uppercase; letter-spacing: 0.1em; margin-bottom: 0.5rem;
  }

  /* Collapsible mini-report */
  .mini-report { border: none; background: none; width: 100%; }
  .mini-report summary {
    font-size: 0.8rem; color: #333; cursor: pointer; font-weight: 600;
    padding: 0.4rem 0; list-style: none;
    display: flex; align-items: center; gap: 0.5rem;
  }
  .mini-report summary::-webkit-details-marker { display: none; }
  .mini-report summary::before {
    content: '▶'; font-size: 0.6rem; color: #2c5f2d; transition: transform 0.15s;
  }
  .mini-report[open] summary::before { transform: rotate(90deg); }

  /* Need-to-read score badges */
  .need-score {
    display: inline-block; padding: 0.1rem 0.4rem; border-radius: 3px;
    font-size: 0.7rem; font-weight: 700;
  }
  .need-1, .need-2 { background: #e8f5e9; color: #2e7d32; }
  .need-3 { background: #fff8e1; color: #f57f17; }
  .need-4 { background: #fff3e0; color: #e65100; }
  .need-5 { background: #fce4ec; color: #c62828; }

  /* Bridging entries */
  .entry {
    margin-bottom: 0.75rem; padding-bottom: 0.75rem;
    border-bottom: 1px solid #d5e8d5;
  }
  .entry:last-child { margin-bottom: 0; padding-bottom: 0; border-bottom: none; }
  .entry-header {
    display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.3rem;
  }
  .entry-name { font-size: 0.88rem; font-weight: 700; color: #2c5f2d; }
  .entry-type {
    font-size: 0.6rem; color: #888; text-transform: uppercase;
    letter-spacing: 0.05em; white-space: nowrap;
  }
  .scaffold {
    font-size: 0.82rem; color: #333; line-height: 1.55; margin-bottom: 0.4rem;
  }

  /* Depth dropdown */
  .depth-toggle { margin-bottom: 0.3rem; }
  .depth-toggle summary {
    font-size: 0.75rem; color: #2c5f2d; cursor: pointer; font-weight: 600;
  }
  .depth-content {
    font-size: 0.78rem; color: #555; line-height: 1.55;
    border-left: 2px solid #d5e8d5; padding-left: 0.6rem; margin-top: 0.2rem;
  }

  /* Feedback buttons */
  .feedback-buttons { display: flex; gap: 0.3rem; margin-top: 0.35rem; }
  .feedback-buttons button {
    font-size: 0.62rem; padding: 0.2rem 0.5rem; border-radius: 3px;
    border: 1px solid #ccc; background: #fff; cursor: pointer; color: #666;
  }
  .feedback-buttons button:hover { border-color: #2c5f2d; color: #2c5f2d; }
  .feedback-buttons button.active-a { background: #fee; border-color: #c66; color: #c66; }
  .feedback-buttons button.active-b { background: #fef8e6; border-color: #c90; color: #b80; }
  .feedback-buttons button.active-c { background: #efd; border-color: #2c5f2d; color: #2c5f2d; }

  /* Highlighted terms in article text */
  .gargon-term {
    background: linear-gradient(to bottom, transparent 60%, #c8e6c9 60%);
    padding: 0 1px; cursor: help; border-bottom: 1px dotted #2c5f2d;
  }
  .gargon-term:hover { background: #c8e6c9; }

  /* Sticky tone toggle */
  .tone-toggle-wrap {
    position: fixed; top: 1rem; right: 1.5rem; z-index: 1000;
    background: rgba(250,250,248,0.92); backdrop-filter: blur(6px);
    padding: 0.45rem 0.75rem; border-radius: 6px;
    border: 1px solid #ddd; box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  }
  .tone-toggle {
    display: flex; align-items: center; gap: 0.5rem;
    font-size: 0.7rem; color: #888;
  }
  .tone-switch {
    position: relative; width: 34px; height: 18px; display: inline-block;
  }
  .tone-switch input { opacity: 0; width: 0; height: 0; }
  .tone-slider {
    position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0;
    background: #ccc; border-radius: 18px; transition: 0.25s;
  }
  .tone-slider::before {
    content: ''; position: absolute; height: 14px; width: 14px;
    left: 2px; bottom: 2px; background: #fff; border-radius: 50%; transition: 0.25s;
  }
  .tone-switch input:checked + .tone-slider { background: #c2185b; }
  .tone-switch input:checked + .tone-slider::before { transform: translateX(16px); }
  .tone-mode-label { font-weight: 600; transition: color 0.25s; }
  .tone-mode-label.active-formal { color: #2c5f2d; }
  .tone-mode-label.active-conv { color: #c2185b; }

  /* Conversational mode — green shifts to magenta */
  body.conv-mode .gargon-sidebar { background: #fdf0f5; border-left-color: #c2185b; }
  body.conv-mode .section-label { color: #c2185b; }
  body.conv-mode .entry-name { color: #c2185b; }
  body.conv-mode .mini-report summary::before { color: #c2185b; }
  body.conv-mode .depth-toggle summary { color: #c2185b; }
  body.conv-mode .entry { border-bottom-color: #f0d0e0; }
  body.conv-mode .depth-content { border-left-color: #f0d0e0; }
  body.conv-mode .gargon-term {
    background: linear-gradient(to bottom, transparent 60%, #f8c8dc 60%);
    border-bottom-color: #c2185b;
  }
  body.conv-mode .gargon-term:hover { background: #f8c8dc; }
  body.conv-mode .gargon-acronym-bar span { color: #d4879e; }
  body.conv-mode .gargon-acronym-bar strong { color: #c2185b; }

  /* Acronym bar above sidebar */
  .gargon-acronym-bar {
    max-width: 1200px; margin: 0 auto;
    display: flex; justify-content: flex-end; padding: 0.4rem 0 0.15rem;
  }
  .gargon-acronym-bar span {
    width: 340px; padding-left: 1.25rem;
    font-size: 0.5rem; letter-spacing: 0.12em; color: #5a8a5c;
    text-transform: uppercase;
  }
  .gargon-acronym-bar strong { color: #2c5f2d; font-weight: 700; font-size: 0.55rem; }

  /* Footer */
  .gargon-footer {
    max-width: 1200px; margin: 0 auto; padding: 1.5rem 2rem;
    border-top: 2px solid #2c5f2d; font-size: 0.8rem; color: #888;
  }

  /* Responsive — stack on narrow screens */
  @media (max-width: 900px) {
    .section-row { flex-direction: column; }
    .gargon-sidebar { width: 100%; min-width: unset; border-left: none; border-top: 3px solid #2c5f2d; }
  }
</style>
</head>
<body>

<!-- HEADER: title left, friction heatmap right -->
<div class="gargon-header">
  <div class="header-left">
    <h1>GARGON</h1>
    <div class="article-title">[Article Title]</div>
    <div class="meta">
      <span>Prepared for [User Name]</span>
      <span>[N] bridging concepts</span>
      <span>~[N] min read with context</span>
    </div>
  </div>
  <div class="friction-heatmap">
    <div style="font-size:0.6rem;color:#888;text-transform:uppercase;letter-spacing:0.08em;font-weight:600;margin-bottom:0.3rem;">Friction Map</div>
    <div class="friction-visual">
      <!-- SVG page icon with colored stripes per section -->
      <!-- Stripe height is proportional to section length in the article -->
      <!-- Colors: #ef5350 (high/4-5), #ff9800 (medium-high/4), #fff176 (mid/3), #a5d6a7 (low/2), #c8e6c9 (minimal/1), #e8f5e9 (none/0) -->
      <svg class="friction-doc-svg" width="46" height="72" viewBox="0 0 46 72" xmlns="http://www.w3.org/2000/svg">
        <title>Friction map — top is beginning of article, bottom is end</title>
        <path d="M3 0.5 H34 L45.5 12 V69 Q45.5 71.5 43 71.5 H3 Q0.5 71.5 0.5 69 V3 Q0.5 0.5 3 0.5 Z" fill="#fff" stroke="#bbb" stroke-width="1"/>
        <path d="M34 0.5 V9.5 Q34 12 36.5 12 H45.5" fill="#eee" stroke="#bbb" stroke-width="1"/>
        <!-- Generate one rect per section, y-positioned sequentially, height proportional to section length -->
        <rect x="5" y="5" width="28" height="[proportional]" rx="1" fill="[color]"><title>Section N — Need X/5</title></rect>
        <!-- ... repeat for each section ... -->
      </svg>
      <div class="friction-legend">
        <div class="friction-legend-item"><span class="friction-legend-swatch" style="background:#ef5350"></span> High</div>
        <div class="friction-legend-item"><span class="friction-legend-swatch" style="background:#fff176"></span> Mid</div>
        <div class="friction-legend-item"><span class="friction-legend-swatch" style="background:#a5d6a7"></span> Low</div>
      </div>
    </div>
  </div>
</div>

<!-- STICKY TONE TOGGLE -->
<div class="tone-toggle-wrap">
  <div class="tone-toggle">
    <span class="tone-mode-label active-formal" id="tone-label-standard">Standard</span>
    <label class="tone-switch">
      <input type="checkbox" id="tone-checkbox">
      <span class="tone-slider"></span>
    </label>
    <span class="tone-mode-label" id="tone-label-conv" style="color:#aaa">Plain Speak</span>
  </div>
</div>

<!-- ACRONYM BAR (right-aligned above sidebar) -->
<div class="gargon-acronym-bar"><span><strong>G</strong>uided <strong>A</strong>daptive <strong>R</strong>eduction of <strong>G</strong>aps in <strong>O</strong>perational <strong>N</strong>eeds</span></div>

<!-- SECTION ROW: article panel + sidebar, repeated per section -->
<div class="section-row">
  <div class="gargon-sidebar">
    <div class="section-label">Section N · GARGON Context</div>
    <details class="mini-report">
      <!-- Mini-report starts COLLAPSED — no "open" attribute -->
      <summary>
        [N] concepts: [breakdown by type, e.g. "2 terms, 1 meaning divergence, 1 framework"]
        <span class="report-stats">· Need-to-read <span class="need-score need-[1-5]">[N]/5</span></span>
      </summary>
      <div class="entries">

        <div class="entry">
          <div class="entry-header">
            <span class="entry-name">[Concept Name]</span>
            <span class="entry-type">[term | framework | notation | meaning divergence | cross-domain transfer | recognition gap | analogy]</span>
          </div>
          <!-- data-conv holds the Plain Speak version; JS swaps innerHTML on toggle -->
          <div class="scaffold" data-conv="[Plain Speak version of scaffold]">
            [Standard version of scaffold]
          </div>
          <details class="depth-toggle">
            <summary>More depth</summary>
            <div class="depth-content">
              [3-6 sentence expansion]
            </div>
          </details>
          <div class="feedback-buttons">
            <button>A · Lost</button>
            <button>B · Example</button>
            <button>C · Got it</button>
          </div>
        </div>

        <!-- More entries as needed -->
      </div>
    </details>
  </div>

  <div class="article-panel">
    <!-- Original article text, unchanged EXCEPT for .gargon-term highlights -->
    <p>Article text with <span class="gargon-term" title="GARGON: [Standard tooltip]" data-conv-title="GARGON: [Plain Speak tooltip]">highlighted term</span> inline.</p>
  </div>
</div>

<!-- NO-CONTEXT SECTION -->
<div class="section-row">
  <div class="gargon-sidebar no-context">
    <div class="section-label">Section N</div>
    <div class="no-context-msg">No extra context needed here.</div>
  </div>
  <div class="article-panel">
    [Original article section text]
  </div>
</div>

<!-- FOOTER -->
<div class="gargon-footer">
  Generated by GARGON — Guided Adaptive Reduction of Gaps in Operational Needs<br>
  Profile: [User Name] · [N] bridging concepts across [N] sections · Generated: [Date]
</div>

<script>
// Store formal versions on load
document.querySelectorAll('.scaffold[data-conv]').forEach(el => {
  el.dataset.formal = el.innerHTML;
});

// Tone toggle logic
document.getElementById('tone-checkbox').addEventListener('change', function() {
  const isConv = this.checked;
  document.body.classList.toggle('conv-mode', isConv);

  // Toggle label styling
  document.getElementById('tone-label-standard').className =
    'tone-mode-label' + (isConv ? '' : ' active-formal');
  document.getElementById('tone-label-standard').style.color = isConv ? '#aaa' : '';
  document.getElementById('tone-label-conv').className =
    'tone-mode-label' + (isConv ? ' active-conv' : '');
  document.getElementById('tone-label-conv').style.color = isConv ? '' : '#aaa';

  // Swap scaffold text
  document.querySelectorAll('.scaffold[data-conv]').forEach(el => {
    el.innerHTML = isConv ? el.dataset.conv : el.dataset.formal;
  });

  // Swap highlighted term tooltips
  document.querySelectorAll('.gargon-term[data-conv-title]').forEach(el => {
    if (!el.dataset.formalTitle) el.dataset.formalTitle = el.title;
    el.title = isConv ? el.dataset.convTitle : el.dataset.formalTitle;
  });
});

// Feedback button toggle
document.querySelectorAll('.feedback-buttons button').forEach(btn => {
  btn.addEventListener('click', function() {
    this.parentElement.querySelectorAll('button').forEach(b => {
      b.classList.remove('active-a', 'active-b', 'active-c');
    });
    const type = this.textContent.trim().charAt(0).toLowerCase();
    this.classList.add('active-' + type);
  });
});
</script>

</body>
</html>
```

## Entry Type Labels

Use these labels in the `<span class="entry-type">` tag:

| Detection source | Display label |
|---|---|
| Technical term, acronym, named technology | `term` |
| Meaning divergence (reader knows the wrong meaning) | `meaning divergence` |
| Cross-domain transfer (reader's prior knowledge partially overlaps) | `cross-domain transfer` |
| Recognition without abstraction (word feels known via product/brand) | `recognition gap` |
| Conceptual framework or mental model | `framework` |
| Concept with strong analogy to transfer domain | `analogy` |
| Essential code/math/diagram | `notation` |

## Writing Rules

### Scaffold (Standard mode)
- 1-2 sentences for term bridges, up to 3 for framework bridges
- Informed by processing orientation, motivation, transfer domains
- No jargon not defined elsewhere in the output
- No qualifiers ("simply put," "in layman's terms")
- Active voice, direct language, confident tone

### Scaffold (Plain Speak mode — stored in `data-conv` attribute)
- Same concepts, same accuracy, same epistemic safety
- Lead with the "buddy sentence" — what a smart friend would say out loud
- Shorter, punchier, more conversational
- Same concept count and depth layer content — tone changes, substance doesn't

### Depth expansion
- 3-6 sentences
- What the scaffold left out
- Where the analogy breaks down (if one was used)
- Can introduce one additional term (defined inline)
- Slightly more technical tone

### Highlighted terms in article text
- Every term with a sidebar entry in that section gets a `.gargon-term` span in the article text
- `title` attribute: Standard tooltip (brief, "GARGON: ..." prefix)
- `data-conv-title` attribute: Plain Speak tooltip
- Every highlighted term MUST have a corresponding sidebar entry in that section — no orphan highlights

### Friction heatmap
- SVG page icon in the header, right side
- One colored stripe per section, height proportional to section's share of total article length
- Color mapped to need-to-read score: red (5), orange (4), yellow (3), light green (2), pale green (1), near-white green (0)
- Stripe width narrows slightly for shorter sections to reinforce proportion

### No-context sections
- Sidebar gets `.no-context` class (gray border, muted background)
- Message: "No extra context needed here."
- No mini-report, no entries

## Density Targets

- 2-6 entries per section
- 8-20 total across the full document
- Sections with zero gaps get the no-context treatment
- If total exceeds 20, cut lowest-importance entries
</content>
</invoke>