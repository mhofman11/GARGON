# GARGON — User Mental Model (UMM) Specification

## Version History

- **v1** — Full 7-dimension conceptual architecture
- **v2 (current)** — MVP-scoped architecture reflecting merged consensus (Claude + ChatGPT + Mike). Retains full conceptual model as reference, implements compressed assessment structure.

---

## What the UMM Is

The UMM is a structured estimate of a specific user's **epistemic distance** from the material they're trying to read.

It is not a score, a level, or a ranking. It is a **topology** — a map of peaks, valleys, and bridges that describes what the user knows, what they almost know, how they learn, and what other knowledge they carry that can be leveraged to accelerate understanding.

The UMM is the input that makes everything else in GARGON personalized. Without it, article analysis produces a generic glossary. With it, article analysis produces a targeted, minimal, epistemically honest bridging document.

**Core equation:**

```
UMM estimates: reader position
RATE estimates: text position
GARGON computes: minimum bridge distance
```

**MVP success metric:** Reduce the expected glossary irrelevance rate. The user looks at the sidebar and thinks "these are exactly the things I didn't know" — not "half of these I already know and the other half I don't care about."

---

## MVP UMM Architecture: 3 Assessed Layers + 2 Derived/Contextual Layers

The full conceptual model contains 7 dimensions (preserved in the Reference Architecture below). For MVP, these compress into a 10-15 question assessment (~3 minutes) that captures five layers of signal.

---

### Assessed Layer 1 — Domain Familiarity Vector

**What it maps:** The user's familiarity with terms, concepts, and frameworks across 8 concept clusters within CS/tech.

**Key principle:** Knowledge is not linear. A user may know surface-level AI concepts from reading articles while having no idea how DNS works. The vector captures this unevenness at coarse resolution.

**Concept clusters for CS/Tech MVP:**

1. **Computing fundamentals** — hardware, OS concepts, file systems, memory, networking basics
2. **Programming foundations** — variables, control flow, data types, functions, OOP, debugging
3. **Data & algorithms** — data structures, sorting/searching, complexity, recursion
4. **Web & internet** — HTTP, APIs, DNS, client-server, frontend/backend, databases
5. **DevOps & infrastructure** — containers, CI/CD, cloud services, deployment, monitoring
6. **AI & machine learning** — models, training, inference, transformers, embeddings, fine-tuning, agents
7. **Security & privacy** — encryption, authentication, authorization, vulnerabilities
8. **Software engineering practices** — version control, testing, architecture patterns, documentation

**MVP classification scale (4 levels):**

| Level | Label | Meaning |
|---|---|---|
| 0 | **Unknown** | Never encountered, no frame of reference |
| 1 | **Recognize** | Heard the terms, can't explain them |
| 2 | **Understand** | Can follow an explanation that uses these concepts |
| 3 | **Comfortable** | Can reason with these concepts, apply them, explain them |

**Assessment method:** Combination of self-report ("How familiar are you with...") and recognition questions ("Which of these best describes...") across the 8 clusters. Adaptive branching: if a user marks "Unknown" on a cluster, skip the recognition question for that cluster.

**Note:** The full conceptual model distinguishes 6 knowledge types within each cluster (terms, concepts, frameworks, paradigms, models, importance relationships) and uses a 7-level classification scale. These remain valid for post-MVP expansion but are not required for initial gap filtering.

---

### Assessed Layer 2 — Formal Literacy Profile

**What it maps:** The user's comfort with the notational and symbolic systems used in CS/tech writing, independent of conceptual understanding.

**Why it's in the MVP:** Two people with identical conceptual understanding will have completely different experiences with the same article if one can read a code snippet and the other can't. Formal literacy predicts where *notation* causes friction, not ideas. This is one of the strongest predictors of article-level difficulty.

**Three axes, each rated on a simple scale:**

| Axis | Low | Medium | High |
|---|---|---|---|
| **Code comfort** | Can't parse any code | Can follow pseudocode or simple Python | Reads code in multiple languages |
| **Math notation comfort** | Basic arithmetic only | Comfortable through algebra | Can read statistical/linear algebra notation |
| **Diagram literacy** | Minimal | Can follow flowcharts | Reads architecture diagrams, sequence diagrams, ERDs |

**Assessment method:** 3 questions, one per axis. Can use visual examples ("Can you follow what this snippet does?" with an actual snippet shown).

---

### Assessed Layer 3 — Transfer Domains

**What it maps:** Other fields, hobbies, and professional domains the user knows well — because these create analogy bridges that dramatically accelerate understanding.

**Why it's in the MVP:** A plumber already has strong mental models for networking, data pipelines, and message queues. An auto mechanic understands diagnostics, fault isolation, and system interdependencies — directly transferable to debugging and systems thinking. Transfer domains are the raw material GARGON uses to construct explanations that land instantly.

**MVP implementation:** User selects 2-3 strongest domains from a category list. Tags, not essays.

**Category options:**

- Engineering / trades (mechanical, electrical, plumbing, construction)
- Business / marketing / operations
- Creative fields (design, music, writing, film)
- Academic sciences (biology, chemistry, physics)
- Math-heavy fields (finance, statistics, accounting, engineering)
- Design / architecture / spatial reasoning
- Hands-on systems (repair, troubleshooting, building, tinkering)
- Teaching / coaching / training
- Healthcare / medical
- Legal / compliance / process
- Other (free text)

**What's NOT assessed here (deferred to generation stage):** Explicit analogy mapping between transfer domains and CS concepts. The LLM handles this dynamically at GARGON output time using the domain tags. This keeps assessment lightweight while preserving full analogy capability.

**Latent understanding note:** The concept of "latent knowledge" — things the user would grasp immediately with the right framing because of transfer domain experience — is NOT explicitly assessed. Instead, it exists as an implicit capability of the generation engine. When GARGON produces a bridging definition, it checks the user's transfer domain tags and infers whether an analogy bridge exists. If the user's profile says "plumbing, professional level" and the article discusses load balancing, the system can infer that a flow-distribution analogy will land without needing to have tested for this specifically.

---

### Derived Layer — Confidence Calibration

**What it maps:** The gap between what the user *thinks* they know and what they *actually* know.

**Why it's in the MVP:** This is essentially free signal — no additional questions required. It's computed automatically from the self-report/recognition question pairs already in Layer 1.

**How it works:**

For each concept cluster, Layer 1 already asks both:
- Self-report: "How familiar are you with [cluster]?" (the user's estimate)
- Recognition: "Which of these best describes [concept from that cluster]?" (demonstrated knowledge)

Compare the two across clusters. The pattern reveals calibration:

| Pattern | Classification | GARGON Impact |
|---|---|---|
| Self-report consistently > demonstrated | **Overcalibrated** | Lower the inclusion threshold — surface more definitions than the model alone suggests |
| Self-report consistently < demonstrated | **Undercalibrated** | Raise the threshold — trust the user knows more than they claim |
| Self-report ≈ demonstrated | **Well-calibrated** | Threshold stays as modeled |

**Stored as:** A calibration offset value that adjusts the gap-detection threshold in the GARGON output stage.

---

### Contextual Layer — Motivation & Processing Orientation

**What it maps:** Why the user cares and how they naturally take in new information.

**Why it's in the MVP:** These are 2 questions that meaningfully change *how* bridging content is written, not just *what* gets included. High leverage per question.

**Question 1 — Motivation:** "What most interests you about this topic?"

Options:
- Practical use / building things
- Curiosity / understanding how things work
- Career / professional development
- Staying informed / keeping up
- Self-improvement / learning as a practice
- Specific project or goal (free text)

**Impact:** Shapes output framing. "Builder" motivation → capability-oriented definitions ("this lets you..."). "Curiosity" motivation → explanatory definitions ("this works by..."). "Career" motivation → outcome-oriented definitions ("this matters because...").

**Question 2 — Processing orientation:** "When learning something new, do you prefer..."

- Seeing an example first, then the general principle
- Hearing the concept first, then seeing examples

**Impact:** Determines whether GARGON leads bridging definitions with "Imagine you're..." (concrete-first) or "The general principle is..." (abstract-first).

---

## The Composite MVP UMM Profile

When all layers are populated, the UMM produces a structured profile:

```
User Profile Snapshot
═══════════════════════

Domain familiarity:
  computing fundamentals  → unknown
  programming foundations → recognize
  data & algorithms       → unknown
  web & internet          → understand
  devops & infrastructure → recognize
  AI & machine learning   → recognize
  security & privacy      → unknown
  software engineering    → unknown

Formal literacy:
  code            → low
  math notation   → low-medium
  diagrams        → medium

Transfer strengths:
  engineering/trades (professional)
  hands-on systems (professional)

Confidence calibration:
  slightly undercalibrated (+0.3 offset)

Processing preference:
  example-first

Motivation:
  self-improvement + practical use (AI integration)
```

**Narrative interpretation (for internal system use):**

> Michael has ~10-15% term coverage across CS/tech. Strongest in web concepts (functional from daily use) and AI terminology (surface-level from heavy reading). Zero foundation in algorithms, security, and formal CS theory. Strong transfer potential from professional trades background — mechanical systems and physical flow systems provide natural analogy bridges for debugging, networking, architecture, and pipeline concepts. Low code and notation literacy means articles with code snippets or math will create friction even when the underlying ideas are accessible. Slightly undercalibrated — knows a bit more than he thinks. Reads to build capability and feed self-improvement drive, not for academic understanding. Needs examples before abstractions.

---

## How the UMM Feeds Downstream

| GARGON Stage | What UMM Provides |
|---|---|
| **RATE analysis** | Domain familiarity vector tells the system which article-assumed concepts to flag (only those likely unknown to this user) |
| **Gap detection** | Compares article requirements against the user's actual topology, not a generic difficulty level |
| **Analogy generation** | Transfer domain tags give the LLM an analogy candidate space for constructing bridging explanations |
| **Latent detection** | Transfer domains + domain vector together let the system infer which gaps need one sentence vs full explanation |
| **Output framing** | Processing orientation + motivation shape how definitions are written |
| **Threshold tuning** | Confidence calibration adjusts how aggressively content is surfaced or withheld |
| **Notation handling** | Formal literacy profile determines whether code/math/diagram elements in the article need additional translation |
| **Epistemic honesty** | Motivation + domain level together determine when to flag "this is simplified — the full picture is more complex" |

---

## Assessment Design Summary

**Total questions:** 10-15
**Estimated completion time:** ~3 minutes
**Feels like:** Orientation setup, not evaluation

**Question breakdown:**

| Layer | Questions | Type |
|---|---|---|
| Domain familiarity (self-report) | 8 | "How familiar are you with [cluster]?" — 4-level scale |
| Domain familiarity (recognition) | 3-5 (adaptive) | "Which best describes [concept]?" — only for clusters rated Recognize or above |
| Formal literacy | 3 | One per axis, may include visual examples |
| Transfer domains | 1 | Multi-select from category list + optional free text |
| Processing orientation | 1 | Binary choice |
| Motivation | 1 | Multi-select or single-select from options |

**Adaptive logic:**
- If user marks "Unknown" on a domain cluster → skip the recognition question for that cluster
- If user marks "Unknown" on 6+ clusters → assessment can shorten significantly (most signal already captured)
- Recognition questions are drawn from the *boundary* of the user's claimed knowledge — testing the edges, not the center

**Confidence calibration:** Computed post-assessment, no additional questions.

---

## Full Conceptual Reference Architecture (7 Dimensions)

The MVP compresses the full model. For post-MVP expansion, the complete 7-dimension architecture is:

1. **Domain Knowledge Topology** — full subdomain mapping with 6 knowledge types and 7-level classification (MVP: 8 clusters, 4-level scale)
2. **Transfer Domain Inventory** — domains with depth ratings, conceptual richness scores, and explicit analogy bridge maps (MVP: 2-3 domain tags)
3. **Latent Understanding Layer** — explicit classification of concepts as latent-ready, latent-partial, or not-latent based on transfer domain cross-reference (MVP: deferred to generation-time inference)
4. **Formal Literacy Profile** — 4-axis assessment including symbolic convention familiarity (MVP: 3-axis simplified)
5. **Processing Style** — 3-axis model: concrete/abstract, builder/understander, sequential/structural (MVP: 1 binary question)
6. **Confidence Calibration** — paired self-report/demonstration analysis (MVP: same, derived automatically)
7. **Motivation and Energy Profile** — 4-dimension capture: primary driver, domain relationship, emotional orientation, goal specificity (MVP: 1 multi-select question)

**Post-MVP additions to consider:**
- Misconception layer (detected passively via repeated engagement with "known" concepts)
- Reading history integration (cumulative exposure tracking)
- Knowledge decay modeling (confidence scores that degrade over time, reinforced by re-exposure)
- Feedback loop from GARGON interaction (corrections, dismissals, external lookups)

---

## Open Questions for RATE Phase

1. What signals does RATE need from the UMM to do its job? (This determines whether the UMM resolution is sufficient.)
2. How does the dependency map within an article interact with the user's domain vector? (A concept might be article-critical but user-known, or article-peripheral but user-unknown.)
3. What's the threshold logic for "include in GARGON output" vs "omit"? (This sits at the intersection of UMM and RATE.)
4. How does formal literacy affect RATE extraction? (An article heavy in code snippets needs different treatment than one that's purely prose.)
