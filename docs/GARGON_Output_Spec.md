# GARGON — Guided Adaptive Reduction of Gaps in Operational Needs

## Output Specification

---

## What GARGON Produces

GARGON takes the UMM (User Mental Model) and the RATE (Report and Assessment of Text Examined) and produces an **augmented version of the original article** — the same text, with personalized context blocks inserted before each section.

GARGON is not a separate document the user reads before the article. It is not a glossary appended at the end. It is the article itself, presented side-by-side with personalized context so the right information arrives at the right moment.

**What the user sees (v2 — current):**

```
┌─────────────────────────────────────────────────────────────────────┐
│  GARGON          How to Build Your Second Brain         Friction   │
│                  Prepared for [Name] · 19 concepts      Map [SVG]  │
├─────────────────────────────────────────────────────────────────────┤
│  G·A·R·G·O·N acronym bar                        [Standard ○ Plain]│
├──────────────────────────────────┬──────────────────────────────────┤
│                                  │ Section 1 · GARGON Context      │
│  Original article — Section 1   │ ▶ 8 concepts: 2 terms, ...  4/5 │
│  [text with highlighted terms]   │   [collapsed — click to expand] │
│                                  │                                  │
├──────────────────────────────────┼──────────────────────────────────┤
│                                  │ Section 2                        │
│  Original article — Section 2   │ No extra context needed here.    │
│  [unchanged text]                │                                  │
├──────────────────────────────────┼──────────────────────────────────┤
│  ...continues...                 │ ...continues...                  │
└──────────────────────────────────┴──────────────────────────────────┘
```

**Layout:** Article panel on the left (flexible width), GARGON sidebar on the right (340px fixed). Each section is a horizontal row with both panels. The sidebar contains collapsible mini-reports that start collapsed — the reader clicks to expand entries for any section.

**Key v2 features:**
- Side-by-side layout with article left, GARGON sidebar right
- Sticky tone toggle (Standard / Plain Speak) — follows the user down the page, shifts accent color from green to magenta
- Friction heatmap SVG in header — page icon with colored stripes showing per-section difficulty at a glance
- Highlighted terms in article text with hover tooltips that swap with the tone toggle
- Dual-tone scaffolds — every scaffold has both Standard and Plain Speak versions; same substance, different voice
- Collapsible mini-reports per section with concept count by type and need-to-read score (1-5)
- GARGON acronym bar above the sidebar column
- Responsive — stacks vertically on screens under 900px

---

## The Bridging Entry

A bridging entry is the atomic unit of GARGON output. Each entry addresses one gap detected by the RATE × UMM comparison.

### Entry Structure

Every bridging entry has three layers, but only the first is visible by default:

```
┌─────────────────────────────────────────────────────────┐
│ CONCEPT NAME                                    [A][B][C]│
│                                                          │
│ Functional scaffold (1-2 sentences)                      │
│ The short version. Gets you through the article.         │
│                                                          │
│ ▶ More depth                                             │
│   [collapsed by default — dropdown expansion]            │
│   The leveled-up explanation. More complete picture.     │
│   Nuance, caveats, what we're glossing over.             │
│   The reader sees this exists but doesn't have to        │
│   digest it now.                                         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Layer 1 — Functional scaffold (always visible):**
1-2 sentences. Gets the reader to functional understanding of the concept — enough to continue reading the article without confusion. This is the core product promise: the minimum context that keeps reading continuous.

**Layer 2 — Depth expansion (dropdown, collapsed by default):**
A more complete explanation that acknowledges what the scaffold simplified. This is where epistemic honesty lives. The reader can glance at it to know the full picture exists, skim it for awareness, or read it fully if curious. It solves three problems at once:

- Keeps the visible GARGON output compact (scaffold only)
- Provides epistemic honesty without disclaimers (the dropdown's existence signals "there's more")
- Gives the reader an on-ramp to deeper understanding without forcing it

**Layer 3 — Feedback buttons (always visible, inline with concept name):**

Three buttons next to each entry:

| Button | Label | What it means | Signal type |
|---|---|---|---|
| **A** | Lost | "I don't understand what you're even saying" | GARGON underestimated the gap. UMM needs adjustment downward for this cluster. Entry needs simpler framing or missing prerequisites. |
| **B** | Example | "Give me a quick example" / "Give me another example" | Explanation is in the right zone but didn't land. Presentation issue, not level issue. System generates an additional concrete example on demand. |
| **C** | Got it | "This was exactly what I needed" | GARGON nailed it. Confirms UMM accuracy. Reinforces the model. |

**Feedback loop value:** These buttons are low-friction (one tap, no typing) and generate high-value signal about UMM accuracy, entry quality, and the user's actual comprehension edges. Over time, patterns emerge:

- Consistent A responses in a cluster → UMM overestimates this user's level there
- Consistent B responses → level is right but framing/analogy style needs adjustment
- Consistent C responses → model is well-calibrated, GARGON is working
- A responses on concepts the UMM predicted as "close to known" → the boundary estimate is too generous
- A on a meaning divergence term → common-meaning familiarity is masking technical-meaning confusion

---

## Entry Types

Not all gaps are the same. GARGON uses different entry formats depending on what kind of gap is being bridged.

### Type 1 — Term Bridge

**When used:** A technical term, acronym, or named technology that the user hasn't encountered or can't define.

**Scaffold format:** Direct definition in plain language. 1-2 sentences.

**Example:**

```
┌─────────────────────────────────────────────────────────┐
│ API (Application Programming Interface)         [A][B][C]│
│                                                          │
│ A set of rules that lets one piece of software talk to   │
│ another. When an app "calls an API," it's sending a      │
│ structured request to another system and getting a       │
│ response back.                                           │
│                                                          │
│ ▶ More depth                                             │
│   APIs define what questions you're allowed to ask and   │
│   what format the answer comes back in. They're how      │
│   most modern software is built — separate pieces that   │
│   communicate through these defined interfaces rather    │
│   than being one giant program. REST, GraphQL, and gRPC  │
│   are different styles of API design with different      │
│   tradeoffs.                                             │
└─────────────────────────────────────────────────────────┘
```

### Type 2 — False Confidence Bridge

**When used:** A word the reader believes they understand, but whose meaning diverges from the technical usage, transfers misleadingly from another domain, or was never independently learned (recognition without abstraction). RATE flagged it via the three-source false confidence check.

**Scaffold format:** Explicitly names what the reader likely thinks the word means, then pivots to what it actually means here. This prevents the reader from silently applying the wrong definition — or from not realizing they have no definition at all.

**Example:**

```
┌─────────────────────────────────────────────────────────┐
│ Token (technical meaning)                       [A][B][C]│
│                                                          │
│ You know "token" as a coin substitute or a symbolic      │
│ gesture. In AI/language processing, a token is a chunk   │
│ of text — usually a word or piece of a word — that the   │
│ system processes as a single unit. When you see "token   │
│ limit," it means the maximum amount of text the system   │
│ can handle at once.                                      │
│                                                          │
│ ▶ More depth                                             │
│   Tokenization is how language models break text into    │
│   processable pieces. "Understanding" isn't one token —  │
│   it might be split into "under" + "stand" + "ing."      │
│   Different models tokenize differently. Token counts    │
│   affect cost, speed, and how much context a model can   │
│   consider at once. The token concept also appears in    │
│   security (authentication tokens) with a completely     │
│   different meaning.                                     │
└─────────────────────────────────────────────────────────┘
```

### Type 3 — Framework Bridge

**When used:** The article assumes the reader operates within a conceptual framework or mental model that they likely don't have.

**Scaffold format:** Names the framework, describes what it is in plain terms, and explains why the article is assuming it. These tend to be slightly longer than term bridges because frameworks are structural, not definitional.

**Example:**

```
┌─────────────────────────────────────────────────────────┐
│ Client-Server Model (assumed throughout)         [A][B][C]│
│                                                          │
│ This article assumes you already think about computing   │
│ as a conversation between two roles: a client (the thing │
│ asking for something — your browser, an app) and a       │
│ server (the thing that answers — a computer somewhere    │
│ that processes the request and sends back a response).   │
│ Almost everything in this article involves something     │
│ asking and something answering.                          │
│                                                          │
│ ▶ More depth                                             │
│   The client-server model is one of several ways to      │
│   organize computing. Peer-to-peer is another (where     │
│   every machine is both client and server). Modern       │
│   systems often have many layers of client-server        │
│   relationships — your browser talks to a web server,    │
│   which talks to an API server, which talks to a         │
│   database server. Each layer is a "client" to the       │
│   layer above it and a "server" to the layer below.      │
└─────────────────────────────────────────────────────────┘
```

### Type 4 — Analogy Bridge

**When used:** The concept maps well to something in the user's transfer domains (from UMM). Can be combined with any other type.

**Scaffold format:** Leads with the analogy from the user's known domain, then connects it to the technical concept. The analogy illuminates structure, not detail. 1-2 sentences max for the analogy itself.

**Design rules for analogies:**

- Each analogy is constructed independently — no attempt to build extended metaphors across the document
- If the same transfer domain (e.g., plumbing) is naturally the best fit for multiple concepts, it gets used multiple times. This happens organically, not by design.
- Analogy should illuminate the *structure* of the concept, not replicate its *mechanics*
- When the analogy breaks down (and it will), the depth expansion is where that gets acknowledged

**Example (for a user with plumbing transfer domain):**

```
┌─────────────────────────────────────────────────────────┐
│ Load Balancer                                   [A][B][C]│
│                                                          │
│ Think of a dispatcher routing plumbing jobs to different │
│ crews so no single crew gets overwhelmed. A load         │
│ balancer does the same thing with incoming web requests  │
│ — it distributes them across multiple servers so no      │
│ single one gets overloaded.                              │
│                                                          │
│ ▶ More depth                                             │
│   Load balancers use different strategies to decide      │
│   where to send each request — round-robin (take turns), │
│   least-connections (send to whoever's least busy), or   │
│   weighted (some servers handle more than others). The   │
│   plumbing analogy holds well for the basic concept but  │
│   breaks down at scale: unlike plumbing crews, servers   │
│   can be spun up or shut down automatically based on     │
│   demand. That's called auto-scaling.                    │
└─────────────────────────────────────────────────────────┘
```

### Type 5 — Notation Bridge

**When used:** The article contains a code snippet, math notation, or diagram that's essential to the argument, and the user's formal literacy (from UMM) suggests they can't parse it.

**Scaffold format:** Plain-language translation of what the notation means. Doesn't try to teach the notation — just tells the reader what it *says*.

**Example (for a user with low code comfort):**

```
┌─────────────────────────────────────────────────────────┐
│ Code Block — Section 3 (Python)                 [A][B][C]│
│                                                          │
│ The code below does this: it takes a question from the   │
│ user, searches a database for relevant information,      │
│ combines that information with the question, and sends   │
│ it to an AI model to get an answer. That's the RAG       │
│ pipeline in action — retrieve, then generate.            │
│                                                          │
│ ▶ More depth                                             │
│   Line by line: the first part sets up the connection    │
│   to the database (lines 1-3). Then it converts the      │
│   user's question into a format the database can search  │
│   (line 5 — this is "embedding" the query). Lines 7-8    │
│   search for matching documents. Lines 10-12 combine     │
│   everything into a prompt and send it to the model.     │
│   If you want to understand the code itself, focus on    │
│   the pattern: input → search → combine → output.        │
└─────────────────────────────────────────────────────────┘
```

---

## Context Block Construction

Each GARGON context block (one per article section) is assembled using this logic:

### Step 1 — Gather relevant gaps

From the RATE × UMM gap detection (Stage 1), collect all gaps that are active in this section. This includes:

- Concepts first assumed in this section (new gaps)
- Concepts from earlier sections that this section builds on (carried gaps — only included if the concept is being used in a new way or at a deeper level than before)

### Step 2 — Order by dependency

Within each context block, entries are ordered so that prerequisites appear before the concepts that depend on them. If understanding "embedding" requires understanding "vector," the vector entry comes first.

### Step 3 — Assign entry types

Each gap gets assigned an entry type based on its characteristics:

| Gap characteristic | Entry type assigned |
|---|---|
| Technical term, acronym, or named technology | Type 1 — Term Bridge |
| False confidence term (meaning divergence, cross-domain transfer, or recognition gap) | Type 2 — False Confidence Bridge |
| Conceptual framework or mental model | Type 3 — Framework Bridge |
| Any gap where transfer domain provides strong analogy | Type 4 — Analogy Bridge (combined with the relevant type above) |
| Essential code/math/diagram with low formal literacy match | Type 5 — Notation Bridge |

A single gap can trigger multiple types. A false confidence term with a good plumbing analogy gets a Type 2 + Type 4 hybrid entry.

### Step 4 — Apply density limits

**Target density:** 2-6 entries per context block. 8-20 total across the entire document.

If a section has more gaps than the density limit allows:

- Prioritize by importance score (from RATE Stage 1)
- Concepts that are load-bearing for the article's argument get priority
- Concepts that appear in passing and won't recur can be dropped
- Carried gaps from earlier sections are only re-included if the section uses them in a meaningfully different way

If a section has zero gaps for this user:

- No context block for that section. The reader just reads straight through. This is a good sign — it means the user's knowledge matches the article's assumptions for that section.

### Step 5 — Generate scaffold and depth content

For each entry, generate:

1. The functional scaffold (Layer 1) — informed by:
   - User's processing orientation (example-first → lead with a concrete case; concept-first → lead with the principle)
   - User's motivation (builder → "this lets you..."; curiosity → "this works by..."; career → "this matters because...")
   - User's formal literacy (if low, avoid all jargon in the scaffold itself)
   - Transfer domains (if strong analogy exists, lead with it)

2. The depth expansion (Layer 2) — informed by:
   - What the scaffold simplified or glossed over
   - Where the analogy breaks down (if an analogy was used)
   - Adjacent concepts the reader might encounter next
   - Connections to other concepts in the GARGON output (internal cross-references)

**Scaffold writing rules:**

- Maximum 2 sentences for a term bridge
- Maximum 3 sentences for a framework bridge
- Maximum 2 sentences for the analogy itself (within an analogy bridge)
- No jargon in the scaffold that isn't itself defined elsewhere in the GARGON output
- No qualifiers like "simply put" or "in layman's terms" — these patronize
- Active voice. Direct language. Confident tone.
- The scaffold must be **epistemically safe**: it can simplify, it can omit nuance, but it must not create a mental model that the reader will have to actively unlearn later. "A hash map is like a dictionary where you look things up by word" is safe (it survives further learning). "A hash map is just an array" is not (it plants a misconception).

**Depth expansion writing rules:**

- Can be 3-6 sentences
- Acknowledges what the scaffold left out
- If an analogy was used, notes where it stops mapping accurately
- Can introduce one level of additional terminology (but must define it inline)
- Tone shift: slightly more technical, more precise, less analogical
- Can end with a forward pointer: "This becomes important when you encounter [related concept]"

---

## Tone and Voice

GARGON's voice adapts based on the UMM motivation profile, but stays within these boundaries:

**Always:**
- Confident, not tentative. "This is..." not "This is basically..." or "Simply put..."
- Respectful of the reader's intelligence — they're not stupid, they're just in a different field
- Compact. Every word earns its place.
- Warm without being casual. Helpful without being condescending.

**Motivation-based tone shifts:**

| Motivation | Tone quality | Example phrasing |
|---|---|---|
| Builder / practical use | Capability-oriented | "This is what allows a system to..." / "You'd use this when..." |
| Curiosity / understanding | Explanatory | "This exists because..." / "The reason this matters is..." |
| Career / professional | Outcome-oriented | "This is important in production because..." / "Teams care about this because..." |
| Self-improvement / learning | Growth-oriented | "This connects to..." / "Once you have this, you'll recognize it in..." |
| Staying informed | Contextual | "This is why [thing in the news] works the way it does..." |

These shifts are subtle. The content stays the same; the framing and word choice adjust.

---

## Epistemic Safety Rules

These are non-negotiable design constraints on GARGON output.

**1. No false models.**
A scaffold can simplify. It can omit. It can focus on one aspect of a multi-faceted concept. But it cannot create a mental model that is *wrong* — one that will need to be actively unlearned when the reader eventually encounters the full concept.

Test: "If this reader later becomes an expert, will they have to *undo* what this scaffold taught them, or will they simply *expand* on it?"

If undo → rewrite the scaffold.
If expand → it's safe.

**2. The dropdown is the honesty mechanism.**
The existence of the depth expansion communicates "there's more to this" without saying "we dumbed this down for you." The reader always has access to the fuller picture. They choose when to engage with it.

**3. No false confidence.**
GARGON should not say "this is all you need to know" about any concept. The scaffold is enough *for this article, right now*. The depth expansion gestures toward the larger picture. Neither claims to be complete.

**4. Analogies are bridges, not maps.**
An analogy illuminates one structural similarity. It does not claim the two things are the same. When the analogy breaks down, the depth expansion says where and why. The reader is never left thinking two fundamentally different things are equivalent.

**5. No jargon loops.**
A scaffold cannot use technical terms that aren't themselves defined in the GARGON output. If defining concept A requires concept B, and concept B is also a gap, then B must appear in the context block first (or in an earlier context block). If a scaffold accidentally introduces new jargon, it's broken.

---

## The Feedback System

### Button Mechanics

Each bridging entry has three inline buttons: **A** (Lost), **B** (Example), **C** (Got it).

**Button A — "Lost"**
- User experience: they tap A, the entry immediately regenerates with a simpler explanation. If possible, it adds a prerequisite entry above the current one (the concept the scaffold assumed that it shouldn't have).
- System signal: UMM overestimated this user's knowledge in this cluster. Adjust the domain familiarity vector downward. Log the specific concept as a confirmed gap.
- If A is pressed repeatedly in the same section: the epistemic distance for this article may be larger than estimated. System could surface a note: "This article assumes quite a bit of background in [cluster]. Want me to add more context?" (Not "stop reading" — just "add more anchors.")

**Button B — "Example"**
- User experience: they tap B, a concrete example appears below the scaffold (without replacing it). If an example already exists, a different one is generated.
- System signal: the level is right but the framing didn't land. Log as a presentation issue, not a level issue. Over time, patterns of B presses reveal which *types* of explanations work for this user (more visual? more procedural? more analogical?).

**Button C — "Got it"**
- User experience: the entry can optionally dim or minimize to reduce visual noise on re-reads. The reader has confirmed they have what they need.
- System signal: UMM estimate confirmed accurate for this concept. Reinforces the current calibration. In post-MVP versions with reading history, this concept gets logged as "exposed + confirmed understood" with a timestamp for the knowledge decay model.

### Aggregate Feedback Patterns

Over multiple articles, button patterns reveal:

| Pattern | Inference |
|---|---|
| A concentrated in one cluster | UMM consistently overestimates that cluster |
| B on analogy bridges specifically | User's transfer domain mapping may be weaker than tagged |
| B on term bridges but C on framework bridges | User understands structures but struggles with vocabulary |
| C on everything | UMM is well-calibrated; GARGON is working as intended |
| A on false confidence terms | Common-meaning familiarity or recognition without abstraction is masking gaps (false confidence detection is paying off) |
| No button pressed | Ambiguous — user may be ignoring GARGON, skimming, or finding it adequate without bothering to confirm |

---

## Document Output Format (v2)

GARGON produces a self-contained HTML file. The user pastes an article, GARGON processes it, and returns an augmented HTML document they open in their browser.

### Header:

The header contains three elements in a flex row: title/metadata on the left, friction heatmap on the right.

- **Article title** — what they're about to read
- **Prepared for [name]** — signals personalization (this isn't generic)
- **Bridging concept count** — sets expectations for density
- **Reading time estimate** — with GARGON context
- **Friction heatmap** — an SVG page icon with colored stripes proportional to each section's length, colored by need-to-read score (red = high friction, green = low). This gives the reader a visual preview of where the hardest parts are before they start reading.

### Section rows:

Each section is a flex row with the article panel on the left and the GARGON sidebar on the right. The sidebar starts collapsed — the summary line shows concept count by type (e.g., "6 concepts: 3 terms, 1 meaning divergence, 2 notation") and a color-coded need-to-read badge (1-5). The reader clicks to expand and see the full entries.

### Highlighted terms:

Every term that has a sidebar entry in its section also gets highlighted in the article text with a `.gargon-term` span. Hover shows a tooltip. This creates a visual link between the article and the sidebar — the reader can see at a glance which words GARGON has context for. **Rule: every highlighted term MUST have a corresponding sidebar entry in that section. No orphan highlights.**

### Tone toggle:

A sticky toggle in the top-right corner switches between Standard (green accent, formal scaffolds) and Plain Speak (magenta accent, conversational scaffolds). The toggle follows the user down the page. Switching swaps scaffold text via `data-conv` attributes and highlighted term tooltips via `data-conv-title` attributes. The substance doesn't change — same concepts, same accuracy, same epistemic safety. Only the voice changes.

### Empty section handling:

When a section has no gaps, the sidebar gets a muted style (gray border, lighter background) and shows: "No extra context needed here." This confirms the system is actively monitoring, not just absent.

---

## The Pipeline in Full

```
INPUT:
  User completes UMM assessment (~3 min, one-time)
  User pastes article

STAGE 1 — RATE:
  Pass 1: Extract explicit terminology
  Pass 2: Detect implicit frameworks
  Pass 3: Map prerequisite chains (2-3 levels)
  Pass 4: Detect notation and formalism
  Pass 5: Estimate author's assumed floor
  Pass 6: Align concepts to sections

STAGE 2 — GAP DETECTION (RATE × UMM):
  Compare RATE assumed floor to UMM domain vector → heat map
  Compare RATE terminology to UMM familiarity → term gaps
  Compare RATE frameworks to UMM + transfer domains → framework gaps
  Check false confidence terms → flag regardless of familiarity
  Walk dependency chains → find user's entry points
  Compute importance: criticality × dependency depth
  Apply confidence calibration
  Select top gaps per section (target: 2-6 per section, 8-20 total)

STAGE 3 — GARGON OUTPUT:
  For each section:
    Gather active gaps
    Order by dependency (prerequisites first)
    Assign entry types (term / meaning divergence / cross-domain transfer / recognition gap / framework / analogy / notation)
    Generate Standard scaffolds (informed by processing orientation + motivation + transfer domains + formal literacy)
    Generate Plain Speak scaffolds (same content, conversational voice — stored in data-conv attributes)
    Generate depth expansions
    Attach A/B/C feedback buttons
    Compute need-to-read score (1-5) for section mini-report
  Highlight terms in article text with .gargon-term spans + dual tooltips (title + data-conv-title)
  Generate friction heatmap SVG (per-section colored stripes proportional to section length)
  Assemble side-by-side HTML: article panel left, GARGON sidebar right
  Add header with metadata + friction heatmap
  Add sticky tone toggle, acronym bar
  All mini-reports start collapsed

OUTPUT:
  Self-contained HTML file with side-by-side layout
  Interactive tone toggle (Standard / Plain Speak)
  Interactive feedback buttons per entry
  Friction heatmap visualization
```

---

## Current Scope (v2)

**Implemented:**
- 7 entry types (term, meaning divergence, cross-domain transfer, recognition gap, framework, analogy, notation)
- Side-by-side layout (article left, GARGON sidebar right)
- Collapsible mini-reports with need-to-read scores per section
- Dropdown depth expansion on every entry
- A/B/C feedback buttons on every entry
- Sticky tone toggle (Standard / Plain Speak) with dual-tone scaffolds and tooltips
- Highlighted terms in article text with hover tooltips that swap with tone
- Friction heatmap SVG in header showing per-section difficulty
- GARGON acronym bar above sidebar
- Section-aligned, dependency-ordered entries
- Tone adaptation based on motivation and processing orientation
- Analogy generation from transfer domains
- Notation translation for low-literacy users
- Epistemic safety enforcement (undo vs. expand test)
- Three-source false confidence detection (meaning divergence, cross-domain transfer, recognition without abstraction)
- Responsive layout (stacks on mobile)
- "No extra context needed here" for zero-gap sections

**Simplified for current version:**
- A/B/C signals are visual only — UMM updates happen between articles, not mid-read
- Button B and A are non-functional in the static HTML (they toggle visual state only, establishing the UI pattern)
- Analogies are generated independently each time (no cross-entry coherence tracking)
- Document is static after generation — changing tone is client-side JS, but regenerating content requires a new run

**Future (post-current):**
- Reading history and cross-article learning
- Knowledge decay modeling
- Real-time progressive rendering
- Auto-detection of when to suggest prerequisite reading
- Collaborative features (sharing GARGON profiles, comparing with other users)
- Standalone web app powered by Gemini API (planned — see Product Vision doc)
