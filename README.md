# GARGON

**Guided Adaptive Reduction of Gaps in Operational Needs**

GARGON is a personalized reading comprehension system that identifies what an article assumes you already know, determines what you likely don't know yet, and inserts just enough context to keep you reading continuously — without ever leaving the page.

---

## The Problem

Technical articles assume things about the reader. They assume you know what an API is. They assume you understand the client-server model. They assume you can read a code snippet. When those assumptions are wrong, the reader hits a wall — and the only option is to stop reading, open another tab, look something up, try to hold the new information in working memory, and come back to where they left off.

This breaks flow. It turns a 10-minute read into a 40-minute ordeal. And the constant context-switching makes the material harder to retain.

GARGON solves this by moving key understanding slightly earlier — so the missing context is already there when you need it.

---

## How It Works

GARGON is a three-stage pipeline:

```
UMM (reader position) + RATE (text position) → GARGON (minimum bridge)
```

### Stage 1 — UMM (User Mental Model)

A short assessment (~3 minutes) estimates where you currently sit across the target domain. Not a single score — a topology. Your knowledge has peaks and valleys, and GARGON maps them.

The UMM captures five layers of signal:

- **Domain familiarity** across 8 concept clusters (what you know and don't know across the field)
- **Formal literacy** (can you read code? math notation? architecture diagrams?)
- **Transfer domains** (what other fields you know well — because a plumber already understands data pipelines, they just don't know it yet)
- **Confidence calibration** (the gap between what you think you know and what you actually know — derived automatically, not self-reported)
- **Motivation and processing style** (are you reading to build something or to understand something? do you need examples first or concepts first?)

### Stage 2 — RATE (Report and Assessment of Text Examined)

RATE analyzes any article and extracts everything it assumes the reader already knows. Six extraction passes:

1. **Explicit terminology** — technical terms, acronyms, and named technologies used without definition
2. **Implicit conceptual frameworks** — mental models the author assumes you already think in (this is where "I know all the words but don't understand the paragraph" comes from)
3. **Prerequisite knowledge chains** — recursive dependencies (the article defines "fine-tuning" but assumes you know what "training" means, which assumes you know what a "model" is)
4. **Notation and formalism** — code snippets, math notation, and diagrams, classified by whether they're essential or illustrative
5. **Author's assumed floor** — the baseline knowledge level the author is writing for, estimated per concept cluster
6. **Section-level alignment** — which concepts become relevant at which point in the article

RATE also performs **false confidence detection** across three sources: meaning divergence (common words like "token" or "compile" that carry different technical meanings), cross-domain transfer (words like "schema" that the reader knows from another field but whose meaning partially misleads here), and recognition without abstraction (words like "wiki" that feel known because of product exposure but the reader holds no independent concept). These cause disproportionate confusion because the reader believes they already understand.

### Stage 3 — GARGON Output

GARGON compares the UMM against the RATE output and produces a **self-contained HTML file** with a side-by-side layout: the original article on the left, personalized context in a sidebar on the right.

```
┌──────────────────────────────────┬──────────────────────────────────┐
│                                  │ Section 1 · GARGON Context      │
│  Original article — Section 1   │ ▶ 8 concepts: 2 terms, ...  4/5 │
│  [text with highlighted terms]   │   [click to expand entries]      │
├──────────────────────────────────┼──────────────────────────────────┤
│                                  │ Section 2                        │
│  Original article — Section 2   │ No extra context needed here.    │
│  [unchanged text]                │                                  │
└──────────────────────────────────┴──────────────────────────────────┘
```

The article text is unchanged except for highlighted terms that link to sidebar entries. A sticky tone toggle lets the reader switch between Standard and Plain Speak modes — same substance, different voice. A friction heatmap in the header shows at a glance where the hardest parts of the article are for this reader.

---

## Bridging Entries

Each section's sidebar contains bridging entries — the atomic unit of GARGON output. Entries are grouped in collapsible mini-reports that start collapsed, showing a summary line with concept count by type and a need-to-read score (1-5). The reader clicks to expand.

Every entry has three layers:

**Layer 1 — Functional scaffold** (always visible): 1-2 sentences that get you to functional understanding. Enough to keep reading without confusion. Each scaffold has both a Standard and Plain Speak version — the sticky tone toggle swaps between them.

**Layer 2 — Depth expansion** (dropdown, collapsed by default): The more complete picture. What the scaffold simplified, where the analogy breaks down, what becomes important later. The reader sees this exists — knows the scaffold isn't the whole truth — but doesn't have to engage with it now.

**Layer 3 — Feedback buttons** (A / B / C):

| Button | Meaning |
|---|---|
| **A** — Lost | "I don't understand what you're even saying" |
| **B** — Example | "Give me a quick example" |
| **C** — Got it | "This was exactly what I needed" |

These create a low-friction learning loop. One tap, no typing. Over time, button patterns reveal where the UMM is accurate and where it needs adjustment.

### Entry Types

Not all gaps are the same. GARGON classifies gaps using seven entry types:

| Type | When used | Example |
|---|---|---|
| **Term** | Technical term the user hasn't encountered | *"An API is a set of rules that lets one piece of software talk to another."* |
| **Meaning Divergence** | Reader knows the word but with a different meaning | *"You know 'token' as a coin substitute. In AI, a token is a chunk of text the system processes as a single unit."* |
| **Cross-Domain Transfer** | Reader knows the word from another field; prior knowledge may help or mislead | *"You might know 'schema' from psychology. In tech it means almost the same thing: a file that defines rules and structure."* |
| **Recognition Gap** | Word feels known via product/brand exposure but reader holds no independent concept | *"Not related to Wikipedia specifically. A wiki is an organized set of interlinked pages on a topic."* |
| **Framework** | The article assumes a mental model the user doesn't have | *"This article assumes you think about computing as a conversation between a client (the thing asking) and a server (the thing answering)."* |
| **Analogy** | A concept that maps well to something in the user's transfer domains | *"Think of a load balancer like a dispatcher routing plumbing jobs to different crews so no single crew gets overwhelmed."* |
| **Notation** | Essential code/math/diagram that the user can't parse | *"The code below takes a question, searches a database for relevant info, and sends both to an AI model for an answer."* |

---

## Design Principles

**Epistemic safety.** GARGON can simplify. It can omit nuance. But it cannot plant mental models that the reader will have to unlearn later. The test: "If this reader becomes an expert, will they have to *undo* what this scaffold taught them, or simply *expand* on it?" If undo, rewrite.

**Selection and presentation are separate.** Whether a concept appears in the output is determined by importance (criticality × dependency depth). How it's explained is determined by the user's profile (analogies, examples, framing, notation translation). A concept that's easy to explain is still shown if it's important — it just takes less space.

**No jargon loops.** A scaffold cannot use technical terms that aren't themselves defined elsewhere in the GARGON output. If defining concept A requires concept B, B appears first.

**Density over length.** When epistemic distance is large, GARGON surfaces more prerequisite anchors — more stepping stones, not longer definitions.

**Section alignment preserves flow.** Concepts are surfaced when they become relevant, not front-loaded at the top. A concept from section 4 doesn't appear in the section 1 context block.

---

## Current Scope

**Domain:** Computer science, programming, and tech (chosen because it's the domain where I personally experience this problem most — reading about AI integrations and constantly having to stop to look things up).

**Assessment:** Conversational UMM assessment (~3 minutes). Feels like orientation setup, not evaluation. Confidence calibration derived automatically from the gap between self-report and demonstrated knowledge.

**Output:** Self-contained HTML file with side-by-side layout (article left, GARGON sidebar right). Includes sticky tone toggle (Standard / Plain Speak), friction heatmap, highlighted terms with hover tooltips, collapsible mini-reports per section, and A/B/C feedback buttons per entry.

**Detection:** Three-source false confidence detection — meaning divergence (wrong meaning), cross-domain transfer (partially overlapping prior knowledge), and recognition without abstraction (word feels known via product/brand but reader holds no concept).

**Feedback:** A/B/C buttons establish the UI pattern. UMM updates between articles based on feedback patterns.

**Future:** Reading history that tracks cumulative knowledge across articles. Knowledge decay modeling. Progressive UMM refinement through passive behavioral signals. Standalone web app powered by Gemini API for non-Claude users.

---

## Architecture

```
INPUT
  User completes UMM assessment (~3 min, one-time)
  User pastes article

STAGE 1 — RATE
  Extract explicit terminology
  Detect implicit frameworks
  Map prerequisite chains (2-3 levels deep)
  Detect notation and formalism
  Estimate author's assumed floor
  Align concepts to sections

STAGE 2 — GAP DETECTION (RATE × UMM)
  Compare assumed floor to domain vector → heat map
  Compare terminology to familiarity → term gaps
  Compare frameworks to transfer domains → framework gaps
  Flag false confidence terms regardless of familiarity
  Walk dependency chains → find entry points
  Compute importance = criticality × dependency depth
  Apply confidence calibration
  Select top gaps per section (2-6 per section, 8-20 total)

STAGE 3 — GARGON OUTPUT
  Gather gaps per section
  Order by dependency (prerequisites first)
  Assign entry types (term / meaning divergence / cross-domain transfer / recognition gap / framework / analogy / notation)
  Generate Standard + Plain Speak scaffolds
  Generate depth expansions
  Highlight terms in article text with dual tooltips
  Compute need-to-read scores, generate friction heatmap
  Assemble side-by-side HTML with tone toggle

OUTPUT
  Self-contained HTML file with side-by-side layout
  Sticky tone toggle (Standard / Plain Speak)
  Friction heatmap visualization
  Interactive A/B/C feedback per entry
```

---

## Key Insight

Most reading tools treat difficulty as a property of the text. GARGON treats it as a relationship between the text and the reader.

The same article is easy for one person and impenetrable for another — not because one is smarter, but because they're standing in different places. GARGON measures the **epistemic distance** between where the reader is and where the text assumes they are, then builds the smallest bridge across that gap.

---

## Documentation

| Document | Description |
|---|---|
| [UMM Spec](docs/GARGON_UMM_Spec.md) | User Mental Model — assessment architecture, 5-layer profile structure, composite profile example |
| [RATE Spec](docs/GARGON_RATE_Spec.md) | Report and Assessment of Text Examined — 6 extraction passes, structured footprint format, gap detection pipeline |
| [GARGON Output Spec](docs/GARGON_Output_Spec.md) | Bridging document — entry types, context block construction, epistemic safety rules, feedback system, full pipeline |
| [Architecture Explained](docs/GARGON_Architecture_Explained.md) | Portfolio-ready walkthrough of the three-stage pipeline, false confidence detection, and design decisions |
| [Product Vision](docs/GARGON_Product_Vision.md) | Where GARGON is headed — predictive comprehension, mind-mapping, state transitions, convergence loops |

### Examples

| File | Description |
|---|---|
| [Demo Output (v2)](examples/demo_output_v2.html) | Working GARGON output — open in a browser to see the full side-by-side layout, tone toggle, friction heatmap |
| [Sample Profile](examples/sample_profile.json) | Example UMM profile showing the 5-layer assessment output |

### Claude/Cowork Skill

The [`gargon.skill`](gargon.skill) file is an installable package for [Claude](https://claude.ai) Cowork users. Download it and install in Cowork to use GARGON directly. The skill source lives in [`skill/`](skill/).

---

## Status

v2 complete. Specifications, working Claude skill, and demo HTML output all current. Available as a `.skill` package for Claude/Cowork users. Standalone web app (Gemini-powered) planned.

---

## Live Demo

Open [`examples/demo_output_v2.html`](examples/demo_output_v2.html) in your browser to see a working GARGON output generated for a real article about RAG (Retrieval-Augmented Generation).

---

## Author

Michael Hofman
