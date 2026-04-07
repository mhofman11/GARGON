# UMM Assessment Protocol

## Overview

The UMM assessment builds a 5-layer profile of the reader in ~12 questions. It should feel like a quick orientation chat, not a test.

## Assessment Flow

### Opening

Start conversational:
> "Before I can personalize your reading experience, I need to get a quick sense of where you're at with CS and tech concepts. This isn't a test — there are no wrong answers. I just need to know what's familiar so I know what context to add later."

### Layer 1 — Domain Familiarity Vector (8 questions, adaptive)

For each of the 8 concept clusters, ask a self-report question followed by a recognition question (if applicable).

**The 8 clusters:**

1. Computing fundamentals (hardware, OS, file systems, memory, networking)
2. Programming foundations (variables, control flow, data types, functions, OOP, debugging)
3. Data & algorithms (data structures, sorting/searching, complexity, recursion)
4. Web & internet (HTTP, APIs, DNS, client-server, frontend/backend, databases)
5. DevOps & infrastructure (containers, CI/CD, cloud services, deployment, monitoring)
6. AI & machine learning (models, training, inference, transformers, embeddings, agents)
7. Security & privacy (encryption, authentication, authorization, vulnerabilities)
8. Software engineering practices (version control, testing, architecture patterns)

**Self-report question format:**
> "How familiar are you with [cluster description]? For example, things like [2-3 example concepts from cluster]."
>
> Options: Never encountered / Heard the terms / Can follow an explanation / Comfortable reasoning with these

**The 4-level scale:**
- **Unknown** (0) — never encountered
- **Recognize** (1) — heard the terms, can't explain
- **Understand** (2) — can follow an explanation that uses these concepts
- **Comfortable** (3) — can reason with, apply, and explain these concepts

**Recognition question format (only for clusters rated Recognize or above):**
> "Quick check — which of these best describes [specific concept from that cluster]?"
> Give 3-4 options: one correct, one common misconception, one too vague, one too specific.

**Adaptive branching:**
- If user says "Unknown" → skip the recognition question, move to next cluster
- If user says "Unknown" on 6+ clusters → you have strong signal. You can abbreviate remaining clusters: "It sounds like most of the deeper technical stuff is new territory. Just to round out the picture — are any of these areas ones you've had some exposure to? [list remaining clusters briefly]"

**Confidence calibration (derived, not asked):**
After you have both self-report and recognition answers, compare them:
- Self-report consistently higher than demonstrated → overcalibrated (offset: surface more definitions)
- Self-report consistently lower than demonstrated → undercalibrated (offset: trust user knows more)
- Roughly aligned → well-calibrated (no adjustment needed)

Store as a calibration value: -1 (undercalibrated), 0 (well-calibrated), +1 (overcalibrated)

### Layer 2 — Formal Literacy (3 questions)

**Code comfort:**
> "If you saw a short code snippet in an article — say 5-10 lines of Python or JavaScript — what would your experience be?"
> - Can't parse any code at all
> - Can follow pseudocode or very simple examples with comments
> - Can read code in at least one language
> - Comfortable reading code in multiple languages

**Math notation:**
> "How about math notation? If an article included a formula or equation, how would that go?"
> - Basic arithmetic is fine, anything beyond that is a wall
> - Comfortable through algebra
> - Can handle statistical notation or linear algebra concepts
> - Fluent with advanced mathematical notation

**Diagram literacy:**
> "And diagrams — things like flowcharts, architecture diagrams, system diagrams?"
> - Not really my thing
> - Can follow basic flowcharts
> - Comfortable with most technical diagrams
> - Can read and reason about complex architecture/system diagrams

### Layer 3 — Transfer Domains (1 question)

> "What do you do outside of tech? What fields do you know well — professionally, as hobbies, or just from life experience? These help me find better analogies when explaining technical concepts."

Let them answer freely. Then categorize into:
- Engineering / trades (mechanical, electrical, plumbing, construction)
- Business / marketing / operations
- Creative fields (design, music, writing, film)
- Academic sciences (biology, chemistry, physics)
- Math-heavy fields (finance, statistics, accounting)
- Design / architecture / spatial reasoning
- Hands-on systems (repair, troubleshooting, tinkering)
- Teaching / coaching / training
- Healthcare / medical
- Legal / compliance / process

Tag the top 2-3 with rough depth (hobbyist / experienced / professional / expert).

### Layer 4 — Motivation (1 question)

> "Last couple questions. What's drawing you to this kind of reading? What's the goal?"

Map their answer to one or more of:
- Practical use / building things
- Curiosity / understanding how things work
- Career / professional development
- Staying informed / keeping up
- Self-improvement / learning as a practice
- Specific project or goal

### Layer 5 — Processing Orientation (1 question)

> "When you're learning something new, do you prefer seeing a concrete example first and then hearing the general principle, or do you prefer getting the concept first and then seeing examples?"

Binary: example-first or concept-first.

---

## Profile JSON Structure

Save the completed profile as `gargon_profile.json`:

```json
{
  "name": "[User Name]",
  "created": "[date]",
  "last_updated": "[date]",
  "domain_familiarity": {
    "computing_fundamentals": 0-3,
    "programming_foundations": 0-3,
    "data_algorithms": 0-3,
    "web_internet": 0-3,
    "devops_infrastructure": 0-3,
    "ai_ml": 0-3,
    "security_privacy": 0-3,
    "software_engineering": 0-3
  },
  "formal_literacy": {
    "code": "none | low | medium | high",
    "math_notation": "none | low | low-medium | medium | high",
    "diagrams": "none | low | medium | high"
  },
  "transfer_domains": [
    {"domain": "[field]", "depth": "hobbyist | experienced | professional | expert"}
  ],
  "motivation": ["practical-use | curiosity | career | staying-informed | self-improvement | specific-project"],
  "processing_orientation": "example-first | concept-first",
  "confidence_calibration": -1 | 0 | +1,
  "feedback_history": []
}
```

---

## Quick Recalibration Protocol

For returning users who choose "quick recalibration":

1. Check if there's any A/B/C feedback history. If so, focus recalibration on clusters where A (Lost) signals concentrated.
2. Ask 3-4 questions:
   - "Has anything changed in what you've been reading or learning about lately?"
   - "Any areas where you've been picking up more knowledge since last time?"
   - 1-2 targeted recognition questions for borderline clusters
3. Update the profile and save.
