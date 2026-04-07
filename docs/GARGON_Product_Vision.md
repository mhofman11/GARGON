# GARGON — What the System Becomes

---

GARGON starts as a reading aid. It becomes something else.

On first use, the system takes a rough measurement of what the reader knows. It maps their position in knowledge space — which technical clusters they're comfortable in, which ones they only recognize, where their confidence outpaces their comprehension. Then it runs that map against an article and produces just enough bridging content to keep them reading.

That's useful. But it's not the product.

---

## The System Is Predictive, Not Reactive

Most reading tools wait. The reader hits a word they don't know, pauses, looks it up, loses their place, returns to the article slightly disoriented. GARGON inverts this. It surfaces gaps before the reader encounters them — and more importantly, it surfaces gaps the reader doesn't know they have.

Recognition without abstraction is the clearest proof. During live testing, the reader passed over "wiki" without hesitation. They'd heard it a thousand times inside "Wikipedia." They had no idea they couldn't define it independently. They never would have asked. The system predicted the confusion before it was felt, because the reader's own self-model couldn't see it.

This is a fundamentally different product category. A glossary answers questions. A tutor responds to confusion. GARGON anticipates both — including the confusion the reader can't articulate.

---

## The System Maps the Reader, Not Just the Text

RATE maps the article's conceptual demands. UMM maps the reader's knowledge state. GARGON operates on the delta between them.

This means the output isn't shaped by what's hard — it's shaped by what's hard *for this person*. The same article produces different GARGON output for different readers. A reader with a psychology background gets a cross-domain transfer note on "schema" that leverages what they already know. A reader without that background gets a standalone explanation. The bridging content is a function of who's reading, not just what's written.

The AI serves as the diagnostic instrument. It conducts the UMM assessment, probes the reader's knowledge boundaries, observes the gap between what they claim and what they demonstrate, and derives a confidence calibration the reader never sees. The reader can't perform this measurement on themselves — that's the whole problem with self-reported knowledge. The AI does what the reader literally cannot: build an honest model of their own blind spots.

---

## State Transition as the Core Abstraction

GARGON is fundamentally a state management system. The reader occupies a position in contextual space — a high-dimensional representation of what they know, how they learn, and where their confidence is miscalibrated. The article assumes a different position. GARGON computes the minimum intervention to move the reader from their current state to the state the article requires.

This resembles a pindrop in contextual space, but the dimensions aren't uniform. Some gaps are invisible to the reader. Some are self-correcting. Some cascade — miss one and the next five paragraphs collapse. GARGON doesn't just measure the distance between two points. It identifies which dimensions are load-bearing for this specific traversal, and intervenes only there.

The output is a targeted state transition. Not general education. Not comprehensive coverage. The minimum viable shift from "can't read this article" to "can read this article" — and nothing more.

---

## The Convergence Loop

The first assessment is a ballpark. The system takes the best measurement it can from a short interaction and works with that. The output is useful but coarse.

Then the reader finishes the article and the feedback comes back. The A/B/C buttons on each bridging entry — Lost, Example, Got it — aren't just UX. They're recalibration data. Every response adjusts the confidence offset, refines the cluster familiarity estimates, and sharpens the model.

By the second article, the system already knows more than the initial assessment captured. By the fifth, it has something the assessment never could have produced: a trajectory. Not just where the reader is in knowledge space, but which direction they're moving and how fast. That's momentum — the rate of change of the pindrop across articles.

This is where the product shifts. The first use is a reading aid. Repeated use becomes a progressively higher-resolution map of a specific reader's knowledge topology. The AI records the mind — not as a static snapshot, but as an evolving model that tightens with every interaction.

The assessment starts the loop. GARGON's output provides the bridging content. The reader's feedback refines the model. The refined model produces better output on the next article. Each cycle reduces the gap between the system's model and the reader's actual state. The system converges on the reader.

---

## What This Points Toward

A single GARGON-augmented article is a product. The convergence loop is a platform.

Once the system has a high-fidelity model of a reader's knowledge state and learning trajectory, the applications extend beyond reading comprehension. The same model that predicts "you'll get stuck on tokens in paragraph 4" could predict "you're ready for this article but not that one," or "your understanding of state management has been growing — here's the next concept that would unlock a cluster of related ideas."

The UMM becomes a portable knowledge passport. The reader carries it across articles, across domains, across time. The system doesn't just help them read — it tracks what they've learned, what they're learning, and what they're ready to learn next.

GARGON starts as a reading aid. It becomes a map of the reader's mind that gets sharper every time they use it.

---

*Designed by Michael Hofman, 2026.*
*Architecture developed through iterative specification with Claude (Anthropic) and reviewed by ChatGPT (OpenAI).*
