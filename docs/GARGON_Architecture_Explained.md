# The Architecture Behind GARGON

---

GARGON is a system that reads a technical article, figures out what a specific reader doesn't know, and produces an augmented version with just enough context to keep them moving.

AI was instrumental in building it. The implementation — HTML, CSS, JavaScript, the skill logic — moved quickly with AI handling the code generation. But the architecture upstream of the code is where the project spent most of its time, because that's where the decisions live that determine whether the output actually works or just looks like it does.

This document walks through those decisions.

---

## Three Systems, Not One

GARGON is three independent systems with defined interfaces between them.

**UMM** (User Mental Model) profiles the reader. It produces a structured JSON object describing what this person knows, how they learn, what adjacent domains they can draw from, and how calibrated their self-assessment is.

**RATE** (Report and Assessment of Text Examined) analyzes the article. It extracts every concept the author assumes the reader already understands — terminology, frameworks, notation, prerequisite chains — without knowing anything about who will read it.

**GARGON** (the output engine) computes the gap between the two and generates bridging content — the minimum context needed for this specific reader to get through this specific article.

The separation means the reader profile is portable across articles, the article analysis is independent of who's reading, and the output is a function of the delta between them. Improve any one system and the others stay valid.

---

## Confidence Calibration Is Derived, Not Declared

During the UMM assessment, the reader never gets asked "how confident are you?" Instead, the system compares what the reader claims to know with what they demonstrate during the assessment.

Someone who says they "understand" a cluster but gives vague definitions gets a negative calibration offset — GARGON flags more concepts than the self-report alone would suggest. Someone who says they only "recognize" a cluster but gives surprisingly accurate descriptions gets a positive offset.

This matters because self-reported confidence is unreliable in both directions, and the correction happens without the reader knowing it's there.

---

## Selection and Presentation Are Separate Stages

The gap detection pipeline has two stages with different inputs.

**Selection** answers: "Which concepts does this reader need bridging on?" It uses criticality, dependency depth, and concept reuse density — properties of how important this concept is to the article, crossed with the reader's profile.

**Presentation** answers: "How should this bridge be built?" It uses the reader's formal literacy, processing orientation, transfer domains, and motivation — properties of how this person learns.

The split prevents the system from conflating "importance" with "explainability." A concept that's critical but hard to analogize still gets surfaced. Without the split, hard-to-explain concepts would quietly get deprioritized — which is exactly backwards.

---

## Three Sources of False Confidence

Most reading tools check for one thing: does this word mean something different in technical vs. everyday usage? GARGON detects three distinct sources of miscomprehension, each with a different mechanism.

**Meaning divergence.** The word has an everyday meaning and the article uses a different technical one. The reader genuinely understands the word — just the wrong version of it. "Compile" means gather in English, translate code in CS. The reader's prior knowledge is real but misdirected.

**Cross-domain transfer.** The reader knows the word from a different field, and the meaning partially overlaps. "Schema" feels familiar from psychology, and that intuition isn't entirely wrong — both usages involve organized structure. This source is unique because the reader's prior knowledge can be a hazard (if the overlap misleads) or a bridge (if the overlap provides structural intuition). GARGON leverages the bridge when it exists.

**Recognition without abstraction.** The word feels known because it lives inside a product name the reader recognizes, but they've never formed an independent concept for it. "Wiki" feels known because of Wikipedia — but ask someone to define "wiki" without referencing Wikipedia and most people can't. This is not polysemy. The reader doesn't know a wrong meaning — they hold no meaning at all while believing they do.

The third source is the most interesting because it's invisible from both sides. The reader won't flag it as a gap. The confidence calibration can't catch it. A standard article analysis would skip it. It can only be caught by the RATE side asking: "Would this reader define this word without referencing the product they know it from?"

This category was discovered during live testing when the reader reported that "wiki" created confusion they couldn't articulate — they knew the word but didn't actually know what it meant. That moment led to a structural change in the detection framework.

---

## Epistemic Safety as a Hard Constraint

Every scaffold must pass the "undo vs. expand" test. If the reader later learns the full version of this concept, will they need to unlearn what GARGON told them, or simply expand on it?

If unlearn — the scaffold gets rewritten.

This prevents the most common failure mode in simplified explanations: planting false mental models that feel helpful now but create conceptual debt later. The dropdown depth layer signals "there's more here" without the scaffold itself needing to oversimplify.

---

## Transfer Domain Bridging

The UMM captures the reader's non-technical domains — hobbies, professional background, casual expertise. GARGON uses these to build analogies that are structurally accurate, not just rhetorically appealing.

When the system explains error compounding to a reader whose background is personal training, the analogy maps directly: logging your squat wrong means every future training cycle builds on a bad number. Progressive overload *is* compounding — the domains differ, the mechanism is identical. The reader can reason forward from what they already know rather than accepting a metaphor on faith.

---

## Tone as a Presentation Variable

GARGON offers Standard and Plain Speak modes. The toggle changes sentence structure, formality, and where the payoff lands in the explanation. It changes nothing about which concepts appear, what the depth layer says, or any structural property of the output.

The same bridge can be delivered as "A schema defines the structure and rules governing how information is organized" or "It's the instruction manual for the AI." Both are accurate. Neither plants a false model. The reader picks the voice that resolves their confusion faster.

This also serves as a validation of the two-stage pipeline — if selection and presentation weren't genuinely separate, changing the tone would risk changing what gets surfaced. It doesn't.

---

## Density Limits

GARGON caps bridging entries at 2–6 per section and 8–20 total. A reader who needs 15 concepts explained in a single section doesn't need more explanations — they need an easier article. The cap forces the selection algorithm to prioritize: only the concepts that, if missed, would make the article break down.

---

## How the Systems Interact

```
UMM Assessment → reader profile (JSON, persistent)
         ↓
RATE Analysis → article concept map
         ↓
Gap Detection → reader profile × concept map = prioritized gap list
         ↓
GARGON Output → bridges per gap, shaped by reader profile
         ↓
Feedback (A/B/C buttons) → profile refinement
```

The reader profile persists across articles. Returning readers can reuse their profile, do a quick recalibration, or start fresh. Feedback from each article adjusts the calibration offset — the system's model of the reader sharpens with every use.

---

## Summary

The implementation moved fast because AI is good at generating code from clear specifications. The specifications themselves — the system boundaries, the detection layers, the safety constraints, the interaction model — are where the design work happened. Each decision addresses a specific way the system could produce output that looks helpful but quietly fails the reader.

The question GARGON answers isn't "can you explain the hard words?" It's "how do you figure out what someone doesn't know — including the things they don't realize they don't know — and give them exactly enough context to keep reading?"

---

*Designed by Michael Hofman, 2026.*
*Architecture developed through iterative specification with Claude (Anthropic) and reviewed by ChatGPT (OpenAI).*
