# RATE — Report and Assessment of Text Examined

## What RATE Is

RATE is the article analysis engine of the GARGON system. It takes any piece of text and produces a structured report of everything that text **assumes the reader already knows**.

RATE does not summarize the article. It does not evaluate whether the article is good or bad. It answers one question:

**What does this text require from a reader in order to be understood?**

The output of RATE is a structured footprint — a map of the article's conceptual demands. When compared against the UMM (User Mental Model), the gap between them becomes the input for the GARGON bridging document.

**Position in the pipeline:**

```
UMM (reader position) + RATE (text position) → GARGON (minimum bridge)
```

---

## What RATE Extracts

RATE performs six extraction passes on an article. Each pass targets a different layer of assumed knowledge.

---

### Pass 1 — Explicit Terminology

**What it finds:** Technical terms, jargon, acronyms, and named tools/technologies that the article uses without defining.

**Why it matters:** This is the most visible layer of assumed knowledge. When a reader hits a term they don't know, the friction is immediate and obvious.

**What gets extracted:**

| Type | Description | Example |
|---|---|---|
| **Technical terms** | Domain-specific vocabulary used as-is | "containerization," "middleware," "idempotent" |
| **Acronyms** | Abbreviated terms, whether expanded or not | "API," "CI/CD," "LLM," "CRUD" |
| **Named technologies** | Specific tools, frameworks, languages, platforms | "Kubernetes," "React," "PostgreSQL," "Lambda" |
| **Compound terms** | Multi-word phrases that function as a single concept | "dependency injection," "event-driven architecture," "garbage collection" |
| **False confidence terms** | Words the reader believes they understand but whose meaning diverges, transfers misleadingly, or was never independently learned | "model," "token," "state," "function," "pipeline," "wiki" |

**False confidence detection (critical):** Words like "model," "token," "state," and "function" are everyday English words that carry completely different meanings in CS/tech contexts. These cause disproportionate confusion because the reader *thinks* they understand the word — they have a mental referent for it, but it's the wrong one. RATE must flag when a term's technical meaning diverges significantly from its common English meaning, when the reader's cross-domain knowledge partially overlaps but may mislead, or when the reader recognizes the word only through product/brand exposure without holding any independent concept. These terms often need bridging even when the user's domain familiarity vector suggests they "recognize" the cluster, because recognition creates false confidence.

**Extraction logic:**

- Identify all domain-specific terms in the text
- Flag whether each term is **defined** in the article (introduced with explanation) or **assumed** (used without explanation)
- Only assumed terms are candidates for the GARGON output
- Terms that are defined in the article are still logged (they may depend on other assumed terms)
- For each term, run the **three-source false confidence check:**
  1. **Meaning divergence:** Does this word have a common non-technical meaning that differs from the technical usage? (e.g., "compile" = gather vs. translate code; "token" = coin vs. AI text unit). If yes → flag, requires explicit bridging regardless of cluster familiarity. The reader knows a real meaning — it's just the wrong one.
  2. **Cross-domain transfer:** Does the reader likely know this word from a different field where the meaning partially overlaps? (e.g., "schema" known from psychology but used here as a file format). If yes → flag, because the reader's reference point may mislead — but note that it may also serve as a productive bridge if the structural overlap is genuine.
  3. **Recognition without abstraction:** Does this word feel known because it's embedded inside a well-known product name, brand, or phrase — but the reader has likely never encountered the word as an independent concept? Test: "Would the reader confidently define this word *without* referencing the product they know it from?" If no → flag. This is the most dangerous category because the reader won't self-report it as a gap and confidence calibration won't catch it. It must be detected on the RATE side. This is not polysemy — the reader holds no meaning at all, only the illusion of familiarity. (e.g., "wiki" feels known because of Wikipedia, but the reader may have no idea what a wiki actually is as a standalone concept; "bluetooth" feels known from headphones but the reader may not know it's a wireless protocol standard.)

**Key distinction:** A term that appears in the article with a definition ("An API, or Application Programming Interface, is...") is treated differently from one that's dropped without ceremony ("...the API returns a JSON payload"). The first is the author teaching; the second is the author assuming.

---

### Pass 2 — Implicit Conceptual Frameworks

**What it finds:** Mental models, paradigms, and ways of thinking that the article assumes the reader already operates within — even if no single term signals this.

**Why it matters:** This is the hardest layer to detect and the most damaging when missed. A reader can know every term in an article and still be lost because they don't share the author's underlying framework. This is where "I know all the words but I don't understand the paragraph" comes from.

**Examples:**

| Framework assumed | How it manifests in text | What the reader needs to already understand |
|---|---|---|
| Client-server model | "When the client sends a request..." | That computing often involves two distinct roles: one asks, one answers |
| Abstraction layers | "At the application layer..." | That systems are organized in stacked levels that hide complexity |
| State management | "The component re-renders when state changes..." | That software tracks changing information and responds to changes |
| Asynchronous thinking | "The callback fires when the response arrives..." | That operations can happen out of order and the system handles this |
| Version control mental model | "Merge the feature branch into main..." | That code exists in parallel timelines that can be combined |

**Extraction logic:**

- Identify passages where the author's reasoning depends on a framework that is never introduced
- Classify the framework by type: architectural pattern, computational paradigm, design philosophy, systems model
- Assess how deeply the article relies on this framework: is it a passing reference or is it foundational to the entire argument?
- Flag the framework's **depth of assumption**: does the article assume surface familiarity ("you've heard of this") or operational fluency ("you think in these terms")?

---

### Pass 3 — Prerequisite Knowledge Chains

**What it finds:** Concepts that the article introduces or discusses that themselves depend on prior knowledge the article doesn't provide.

**Why it matters:** An article might helpfully define "fine-tuning" — but if the reader doesn't know what a "model" is in the ML sense, or what "training" means, the definition is useless. RATE needs to detect not just what the article assumes, but what the article's *explanations* assume.

**This is recursive:** The prerequisites have prerequisites. RATE needs to go at least 2-3 levels deep to be useful.

**Example chain:**

```
Article discusses: "fine-tuning a pre-trained model"
  └── Assumes: you know what "fine-tuning" means
       └── Requires: understanding of "training" (adjusting model weights)
            └── Requires: understanding of "model" (in the ML sense)
                 └── Requires: understanding of "parameters" / "weights"
  └── Assumes: you know what "pre-trained" means
       └── Requires: understanding that models learn from data
            └── Requires: basic concept of "dataset"
```

**Extraction logic:**

- For each assumed term or concept, identify its direct prerequisites
- Recurse to a depth of 2-3 levels (MVP ceiling — deeper chains are diminishing returns)
- Store as a directed graph: concept → depends on → [concept, concept, ...]
- Flag the **entry point** — the deepest prerequisite in the chain that the user would need to start from

---

### Pass 4 — Notation and Formalism Detection

**What it finds:** The formal languages, symbolic systems, and visual conventions the article uses.

**Why it matters:** This maps directly to the UMM's Formal Literacy Profile. An article that's conceptually accessible can still be impenetrable if it's expressed in code, math notation, or specialized diagrams the reader can't parse.

**What gets detected:**

| Formalism type | What RATE flags | UMM layer it maps to |
|---|---|---|
| **Code snippets** | Language, complexity level, whether the code is essential to the argument or illustrative | Code comfort |
| **Math notation** | Level (arithmetic → algebra → linear algebra → calculus → statistical), whether it's decorative or structural | Math notation comfort |
| **Diagrams/figures** | Type (flowchart, architecture diagram, sequence diagram, graph/chart), whether understanding depends on the diagram | Diagram literacy |
| **Pseudocode** | Whether present, complexity level | Code comfort (lower bar) |
| **Symbolic conventions** | O(n) notation, type signatures, arrow notation, set notation | Math notation comfort |

**Extraction logic:**

- Scan for code blocks, mathematical expressions, and visual elements
- Classify each by type and complexity
- Assess **criticality**: is this element essential to understanding the article, or is it supplementary? An illustrative code snippet that restates what the prose already said is low-criticality. A code snippet that IS the explanation is high-criticality.
- Output a **formalism profile** for the article that can be directly compared to the UMM's formal literacy layer

---

### Pass 5 — Author's Assumed Floor

**What it finds:** The baseline level of knowledge the author is writing for — the implicit "you must be this tall to ride" of the article.

**Why it matters:** This is a holistic signal, not a per-concept extraction. It tells GARGON how much total bridging work is likely needed and calibrates the overall density of the GARGON output.

**Indicators used to estimate the floor:**

- **Vocabulary density:** Ratio of domain-specific terms to total words. Higher density = higher assumed floor.
- **Definition behavior:** Does the author define basic terms? Intermediate terms? Nothing? The point at which the author stops defining things marks their assumed floor.
- **Sentence complexity:** Average syntactic complexity of explanatory passages. Long nested sentences with multiple embedded concepts signal a higher floor.
- **Reference style:** Does the article cite concepts by name only ("as shown by the CAP theorem") or explain them? Name-only references assume familiarity.
- **Pacing:** How quickly does the article introduce new concepts? Rapid introduction = assumes the reader can absorb quickly because they have nearby scaffolding.

**Output:** A single estimated floor level per UMM concept cluster:

```
Article assumed floor:
  computing fundamentals  → comfortable
  programming foundations → understand
  web & internet          → comfortable
  AI & machine learning   → recognize
  devops & infrastructure → understand
```

**How this is used:** The difference between the article's assumed floor and the user's UMM level per cluster gives a quick "heat map" of where the most friction will occur. Clusters where the gap is largest get the most GARGON attention.

---

### Pass 6 — Section-Level Concept Alignment

**What it finds:** Which concepts become relevant at which point in the article.

**Why it matters:** GARGON's eventual output (even in MVP document form) needs to tell the reader *when* they'll need each piece of context. A concept that first appears in paragraph 8 shouldn't be front-loaded at the top alongside concepts from paragraph 1. Section alignment is what makes the GARGON output feel responsive rather than dumped.

**Extraction logic:**

- Divide the article into logical sections (by headers if available, by topic shift if not)
- For each section, list:
  - New terms/concepts introduced in this section
  - Terms/concepts assumed in this section (used without introduction)
  - Concepts from earlier sections that this section builds on
- Produce a **section map**: an ordered list of which concepts are active in each part of the article

**Output format:**

```
Section 1: "Introduction to RAG"
  New: RAG (defined), retrieval (defined)
  Assumes: LLM, embeddings, vector database
  Builds on: —

Section 2: "How Retrieval Works"
  New: semantic search (defined), cosine similarity (mentioned, not defined)
  Assumes: vectors, embedding space, indexing
  Builds on: embeddings (from Section 1 assumptions)

Section 3: "Implementing RAG with LangChain"
  New: LangChain (named, not explained)
  Assumes: Python, API keys, prompt templates, chaining
  Builds on: RAG (Section 1), retrieval (Section 2)
```

---

## The RATE Output: Structured Footprint

When all six passes complete, RATE produces a single structured object — the article's **conceptual footprint**.

```
RATE Output
═══════════════

Article: [title / first line]
Word count: [n]
Estimated reading time: [n] minutes

─── ASSUMED FLOOR ───
computing fundamentals  → comfortable
programming foundations → understand
web & internet          → comfortable
AI & machine learning   → recognize
devops & infrastructure → understand
security & privacy      → N/A (not addressed)
data & algorithms       → recognize
software engineering    → understand

─── FORMALISM PROFILE ───
code present:     yes — Python, intermediate complexity, high criticality
math notation:    minimal — basic notation only
diagrams:         1 architecture diagram, medium criticality
pseudocode:       none

─── TERMINOLOGY (assumed, not defined) ───
[term]          | cluster              | first appears | criticality
─────────────────────────────────────────────────────────────────
API             | web & internet       | section 1     | high
LLM             | AI & ML              | section 1     | high
embeddings      | AI & ML              | section 1     | high
vector database | AI & ML              | section 1     | high
cosine similarity| data & algorithms   | section 2     | medium
Python          | programming          | section 3     | high
API keys        | web & internet       | section 3     | medium
prompt template | AI & ML              | section 3     | medium

─── IMPLICIT FRAMEWORKS ───
[framework]                 | depth of assumption | sections active
────────────────────────────────────────────────────────────────
client-server model         | operational         | all
vector/embedding space      | surface familiarity | 1, 2
pipeline/chain composition  | surface familiarity | 3

─── DEPENDENCY MAP (top-level chains) ───
embeddings
  └── vectors (math concept)
       └── numerical representation of data
vector database
  └── database (general concept)
  └── embeddings (see above)
  └── indexing / search
cosine similarity
  └── vectors (see above)
  └── angle between vectors (geometric concept)
RAG (defined in article, but definition assumes:)
  └── LLM
  └── embeddings
  └── retrieval vs generation distinction

─── SECTION MAP ───
[section] | new concepts | assumed concepts | builds on
──────────────────────────────────────────────────────
Section 1 | RAG, retrieval | LLM, embeddings, vector DB | —
Section 2 | semantic search | vectors, embedding space, indexing | embeddings
Section 3 | LangChain | Python, API keys, prompt templates, chaining | RAG, retrieval
```

---

## How RATE Interfaces with UMM

The RATE output is designed to be directly comparable to the UMM profile. Each extraction layer has a corresponding UMM layer it maps to:

| RATE Layer | UMM Layer | Comparison Produces |
|---|---|---|
| Assumed floor (per cluster) | Domain familiarity vector | **Heat map**: which clusters have the largest gap between text demands and reader level |
| Formalism profile | Formal literacy profile | **Notation friction**: will the reader struggle with code/math/diagrams in this specific article |
| Terminology list (with clusters) | Domain familiarity vector | **Term gap list**: which specific assumed terms the user likely doesn't know |
| Implicit frameworks | Domain familiarity vector + transfer domains | **Framework gap list**: which mental models the user needs but may not have, and whether transfer domains provide a bridge |
| Dependency map | Domain familiarity vector | **Entry point detection**: how deep down the prerequisite chain does GARGON need to start for this user |
| Section map | (used at output stage) | **Alignment**: when to surface each piece of bridging content |

**The gap detection pipeline (two-stage):**

Important: "Should we show this?" and "How should we explain it?" are separate questions. Stage 1 determines selection. Stage 2 determines presentation. A concept that's easy to explain (high analogy availability) is still shown if it's important — it just takes less space.

**Stage 1 — Selection (what makes the cut):**

This is a RATE × UMM operation.

```
For each assumed concept in RATE:
  1. Check UMM domain familiarity → does the user know this cluster?
  2. If cluster = unknown or recognize → concept is a GAP
  3. If cluster = understand or comfortable → check concept-level familiarity within the cluster
     (user may know the cluster generally but not this specific concept — e.g., "understand" on web cluster doesn't mean they know what a webhook is)
  4. FALSE CONFIDENCE CHECK (three sources):
     a. Meaning divergence → treat as GAP regardless of cluster familiarity (reader knows a real meaning, but it's the wrong one)
     b. Cross-domain transfer → treat as GAP (reader's reference context may mislead — but note potential as analogy bridge)
     c. Recognition without abstraction → treat as GAP even if reader reports confidence (word feels known via product/brand but reader holds no independent concept). **This category is invisible to the UMM** — it must be caught here.
  5. For each GAP:
     a. Check dependency map → does this gap have prerequisites that are also gaps?
     b. Compute importance = criticality × dependency depth
        where criticality = frequency + structural role† + dependency centrality†
        (amplified by concept reuse density across sections)
  6. Apply confidence calibration offset
  7. Rank by importance score
  8. Select top N gaps (target: 5-12 per section, 8-20 total)

  † = LLM-estimated heuristic, not deterministic measurement (see note below)
```

**Stage 2 — Presentation (how it gets explained):**

This is a GARGON output operation, detailed in the GARGON output spec. For each selected gap, presentation is shaped by:

```
For each selected GAP:
  a. Check transfer domains → analogy bridge available?
     → YES: 1-sentence analogy (low explanation cost)
     → PARTIAL: analogy + brief qualifier (medium cost)
     → NO: standalone explanation (higher cost)
  b. Check formalism profile vs formal literacy → is notation a barrier?
     → YES: translate notation into prose
  c. Check processing orientation → example-first or concept-first?
  d. Check motivation → builder framing, curiosity framing, or career framing?
```

**MVP precision note:** Several fields in the selection formula — structural role, dependency centrality, depth of assumption, and concept reuse density — are LLM-estimated heuristics in the MVP, not computed values. They should be treated as useful approximations, not deterministic measurements. The system aims for directionally correct filtering, not ontological completeness. These can be refined with more structured scoring as the system matures.

---

## RATE Design Principles

**1. Assumed, not defined.**
RATE only flags concepts the article assumes. If the article defines something, it's the author's job, not GARGON's — unless the article's definition itself assumes things (which Pass 3 catches).

**2. Criticality matters more than quantity.**
An article might assume 50 terms. Only 8-12 might actually matter for understanding the argument. RATE must distinguish between terms the author uses once in passing and terms that are load-bearing for the article's logic.

Criticality is computed from three signals:
- **Frequency**: how often the concept appears in the article
- **Structural role**: is this concept load-bearing for the article's argument, or decorative?
- **Dependency centrality**: how many other concepts depend on this one? (A concept that sits at the root of multiple dependency chains is higher criticality than one at a leaf)

Additionally, **concept reuse density** acts as a criticality amplifier: a concept that appears across 4+ sections is likely a foundational anchor for the entire article, even if each individual mention seems low-salience. Reuse across sections signals that the author treats the concept as ambient knowledge the reader should carry throughout.

**3. Frameworks are harder to detect but more valuable than terms.**
A reader who doesn't know the term "idempotent" can look it up in 10 seconds. A reader who doesn't understand the client-server model will be lost for the entire article. Framework detection is where RATE provides the most differentiated value.

**4. Depth of assumption varies.**
"Assumes you've heard of Docker" is a different demand than "Assumes you think in terms of containerized microservice architectures." RATE must capture not just *what* is assumed but *how deeply* it's assumed. Surface-level assumptions need lighter bridging than operational-level assumptions.

**5. Recursion has a floor.**
Prerequisite chains can go infinitely deep ("to understand X, you need Y, which requires Z, which requires..."). For MVP, RATE recurses 2-3 levels. The goal is to find the user's **entry point** — the deepest concept in the chain that they'd need to start from — not to map the entire tree of human knowledge.

**6. Section alignment preserves flow.**
The whole point of GARGON is continuous reading. If RATE dumps all concepts into an unstructured list, the GARGON output can't be sequenced properly. Section-level alignment is not optional — it's what makes the output usable.

---

## MVP Scope and Boundaries

**In scope for MVP:**
- All 6 extraction passes
- Structured footprint output
- Direct comparability with UMM format
- Gap detection formula
- Criticality ranking

**Simplified for MVP:**
- Dependency chains limited to 2-3 levels deep
- Framework detection relies on LLM inference rather than a formal ontology
- Section detection uses headers or paragraph breaks (not NLP-based topic modeling)
- Formalism detection is rule-based (regex for code blocks, math notation) plus LLM classification

**Out of scope for MVP:**
- Real-time streaming analysis (RATE runs once per article, not progressively)
- Cross-article concept tracking (that's the post-MVP reading history system)
- Author style analysis (how the author explains things — could improve GARGON output quality but isn't essential)
- Multi-language support

---

## Open Questions for GARGON Output Phase

### Resolved (consensus across reviewers)

1. **Gap count target:** 5-12 items active per section, 8-20 concepts total per article. Better to slightly undershoot than overshoot. Continuity > completeness.
2. **Bridging document structure:** Section-aligned concept groups with dependency-aware ordering inside each section (prerequisites first, then local concepts). Maintains reading flow.
3. **Alternative reading recommendation:** Not for MVP. Introducing a "don't read this" decision adds friction that contradicts the product's purpose. Instead: detect large epistemic distance → surface more prerequisite anchors (more stepping stones, not longer definitions) → let the user proceed. Optional primer suggestions can come later.
4. **Analogy depth:** 1-2 sentences max. Analogy should illuminate structure, not replicate detail. Avoid multi-step analogies in MVP.

### Still open (to be resolved in GARGON output spec)

5. **Epistemic honesty implementation:** How does GARGON signal "this is a useful simplification, the full picture is more complex" without creating anxiety or undermining the reader's confidence? This is a core design question, not a UX detail. Needs more than qualifier tags — it's about the system's relationship with the reader's future understanding.
6. **Bridging tone calibration:** How much does motivation (builder vs curious vs career) change the actual language of definitions? Is this a subtle shift or a meaningfully different output?
7. **Notation translation:** When an article includes a code snippet that's essential to the argument and the user's code comfort is low, does GARGON translate the code into prose? Paraphrase it? Annotate it? This is a RATE × UMM interaction that affects output design.
