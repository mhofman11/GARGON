# RATE Extraction Protocol

## Overview

RATE (Report and Assessment of Text Examined) analyzes an article and extracts everything it assumes the reader already knows. This is an internal analysis — the user never sees the raw RATE output. It feeds directly into gap detection and GARGON output generation.

## The Six Extraction Passes

Perform all six passes on the article. You can think through them in sequence, but the output should be a unified analysis, not six separate reports.

### Pass 1 — Explicit Terminology

Identify every technical term, acronym, named technology, compound term, and false confidence term used in the article.

For each term, classify:
- **Defined in article** — the author introduces and explains it → not a GARGON candidate (but log it — its definition may assume other things)
- **Assumed** — used without explanation → GARGON candidate

**False confidence detection is critical.** Run the three-source check on every term:

**Source 1 — Meaning divergence:** The word has a common everyday meaning AND the article uses it with a different specialized meaning. The reader knows a real meaning — it's just the wrong one. (e.g., "compile" = gather vs. translate code; "token" = coin vs. AI text unit)

**Source 2 — Cross-domain transfer:** The reader likely knows this word from a different field where the meaning partially overlaps. The prior knowledge may mislead OR it may serve as a productive analogy bridge. (e.g., "schema" known from psychology but used here as a file format — the structural intuition partially transfers)

**Source 3 — Recognition without abstraction:** The word feels known because it's embedded inside a well-known product name, brand, or phrase — but the reader has likely never encountered the word as an independent concept. This is not polysemy — the reader holds no meaning at all, only the illusion of familiarity. Test: "Would the reader confidently define this word *without* referencing the product they know it from?" (e.g., "wiki" feels known because of Wikipedia but the reader may have no idea what a wiki actually is; "bluetooth" feels known from headphones but the reader may not know it's a wireless protocol standard)

**Source 3 is the most dangerous** because the reader won't self-report it as a gap and UMM confidence calibration won't catch it. It MUST be detected here on the RATE side. When in doubt, flag it.

Common meaning divergence terms in CS: model, token, state, function, class, object, inheritance, thread, pipeline, stack, queue, tree, branch, fork, merge, container, image, volume, port, socket, shell, kernel, library, framework, argument, parameter, instance, method, property, key, value, hash, table, index, pointer, reference, bug, patch, deploy, build, compile, execute, run.

All three sources need bridging even when the user's familiarity vector suggests they "recognize" the cluster.

### Pass 2 — Implicit Conceptual Frameworks

Look for passages where the author's reasoning depends on a framework never introduced. Signal words and patterns:

- "Since..." / "Because..." / "Given that..." (treats something as established)
- "As we know..." / "Obviously..." / "Of course..." (assumes shared understanding)
- Pronoun references to architectural patterns ("the client," "the server," "the pipeline")
- Comparative statements that assume baseline ("faster than traditional approaches," "unlike monolithic architectures")

For each framework detected, classify:
- **Framework name** — what mental model is being assumed?
- **Depth of assumption** — surface familiarity ("you've heard of this") vs operational fluency ("you think in these terms")
- **Scope** — passing reference or foundational to the entire article?

### Pass 3 — Prerequisite Knowledge Chains

For each assumed term or concept, identify what it itself depends on. Recurse 2-3 levels:

```
fine-tuning (assumed)
  └── training (prerequisite)
       └── model weights (prerequisite)
            └── parameters (prerequisite — this is likely the entry point)
```

The **entry point** is the deepest concept in the chain that the user would need to start from. This is where GARGON's explanation should begin.

Don't go deeper than 3 levels. The goal is to find the user's entry point, not map all of human knowledge.

### Pass 4 — Notation and Formalism

For each code snippet, math expression, or diagram:

- **Type:** code (what language?), math (what level?), diagram (what kind?)
- **Complexity:** simple, intermediate, advanced
- **Criticality:** Is this essential to understanding the article's argument, or is it supplementary/illustrative?
  - High criticality = the code IS the explanation, or the math IS the proof
  - Low criticality = the code restates what the prose already said

### Pass 5 — Author's Assumed Floor

Estimate the baseline knowledge level the author is writing for, per concept cluster:

```
computing fundamentals  → [unknown/recognize/understand/comfortable]
programming foundations → [...]
data & algorithms       → [...]
web & internet          → [...]
devops & infrastructure → [...]
ai & machine learning   → [...]
security & privacy      → [...]
software engineering    → [...]
```

Use these signals:
- Where does the author stop defining things? (That's their assumed floor.)
- How dense is the jargon? (Higher density = higher floor.)
- How quickly are new concepts introduced? (Rapid pacing = assumes existing scaffolding.)
- Does the author use name-only references? ("as the CAP theorem shows" vs "the CAP theorem, which states that...")

### Pass 6 — Section-Level Alignment

Divide the article into logical sections (by headers if available, by topic shift if not).

For each section, map:
- New concepts introduced in this section
- Concepts assumed (used without introduction)
- Concepts from earlier sections this section builds on

This alignment drives the GARGON context block placement — each block should contain only concepts relevant to the upcoming section.

---

## Gap Detection

After completing all six passes, compare against the user's UMM profile:

**For each assumed concept:**

1. Look up the user's familiarity level for the relevant cluster
2. If **unknown** or **recognize** → it's a **GAP**
3. If **understand** or **comfortable** → check whether this specific concept is likely known within that cluster (use judgment — knowing the web cluster at "understand" doesn't mean they know what a webhook is)
4. If the term is flagged by **any false confidence source** (meaning divergence, cross-domain transfer, or recognition without abstraction) → treat as a potential gap regardless of cluster level. **Recognition without abstraction gaps are invisible to the UMM** — they must be caught here.
5. For each gap:
   - Walk the dependency chain — are the prerequisites also gaps?
   - Compute importance: how critical is this concept to the article × how deep is the dependency chain
   - Check if transfer domains provide an analogy bridge
6. Apply confidence calibration offset:
   - Overcalibrated user (+1): include slightly more concepts than importance alone suggests
   - Undercalibrated user (-1): include slightly fewer
7. Rank all gaps by importance
8. Select top gaps per section: target 2-6 per section, 8-20 total for the document

**Important:** A concept that's easy to explain (good analogy available) should NOT be excluded just because it's easy. Importance determines selection. Ease determines presentation length.

Pass the selected gaps, their entry types, their section alignment, and the user's profile to the GARGON output phase.
