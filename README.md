# TakeMeter — Annotated Dataset

This dataset powers **TakeMeter**, a fine-tuned text classifier that measures *discourse quality* on r/ExplainLikeImFive (r/ELI5). Rather than grading a comment as simply "good" or "bad," every comment is tagged with the **type of discourse** it represents. Quality is never labeled directly — it emerges from the mix of types, since a thread full of `reasoning` reads very differently from one full of `reaction`.

For the full taxonomy rationale, edge-case philosophy, and evaluation plan, see [planning.md](planning.md).

---

## Data Sources

All comments were sampled from **r/ELI5** threads. To capture the full range of discourse — not just the polished top answers — comments were drawn from *every* level of each thread, including nested replies. The source threads:

- ELI5: Why are modern vehicles getting more round?
- ELI5: Why does being hit with / submerged in cold water make our breathing sharp and irregular?
- ELI5: Why do big fire engines often accompany ambulances for purely health-related (non-physical-crisis) emergencies?
- ELI5: Why does rationing work? (If I have a glass of water and a single chocolate bar to last 48 hours, what do I gain from spacing them out versus consuming them all at once — is it psychological or physical?)
- ELI5: How do our brains perfectly calculate exactly how much force to use when picking up an object before we even touch it?
- ELI5: Why do flames go upwards if gravity brings things down — and in space, does fire just become a floating blob since no gravity acts on it?
- ELI5: When you "see stars" due to injury or ill health, why is that?
- ELI5: Why do pregnant women start craving some foods more while developing a dislike to others?
- ELI5: What different effects do ADHD medications (Ritalin, Elvanse, Adderall, etc.) have on the brain?
- ELI5: Why does carbon get its own entire branch of chemistry while other elements don't?
- ELI5: Why do we vomit when we have car sickness, sea sickness, etc.?
- ELI5: What really happens when food / water goes down the "wrong pipe"?
- ELI5: Why does China's infrastructure and city development seem so much more advanced than India's, given similar population sizes?

---

## Labeling Process

### The label schema

Each comment is assigned exactly one of four labels. The four types form an implicit quality gradient: `reasoning` and `analogy` are genuine explanations, `assertion` answers without explaining, and `reaction` adds no explanatory content at all.

| Label | Definition |
|---|---|
| `reasoning` | Explains *why* its answer holds using a logical, factual, or mathematical argument. |
| `analogy` | Explains *why* through a comparison, everyday framing, or lived experience rather than a formal argument. |
| `assertion` | States an answer but gives no mechanism — it tells you *that* something is true, not *why*. |
| `reaction` | Makes no explanatory attempt — appreciation, humor, or an emotional response. |

Clarity and completeness were deliberately *not* used as labels: both are too subjective to annotate consistently, so they were tracked only as informal notes.

### Decision rules

Because ELI5 comments routinely blur these categories, two rules were applied consistently across all annotations:

- **Primary explanatory device wins.** When a comment both argues *and* leans on a comparison, strip the comparison away. If a logical argument still stands on its own, it is `reasoning`; if removing the comparison leaves nothing explaining the *why*, the comparison *is* the explanation → `analogy`.
- **Noise floor.** A comment containing *any* genuine explanatory attempt is never `reaction`. The `reaction` label is reserved strictly for comments with no explanatory content.

### Edge cases encountered

Three real comments proved genuinely difficult to label. Documenting them — and the call that was made — keeps the annotation consistent and exposes where the taxonomy is fuzziest.

**1. `reasoning` vs. `analogy` → labeled `analogy`** (from the rationing thread)

> "The also-missed understanding is that rationing isn't about lasting 48 hours from a glass of water and a chocolate bar. It's so you can survive 4 weeks on the provisions you'd otherwise consume in 2–3 weeks."

The commenter uses a time-frame comparison to *carry* their point rather than as decoration. Since the comparison does the explanatory work, the primary-device rule resolves it to `analogy`.

**2. `reasoning` vs. `assertion` → labeled `reasoning`** (water vs. food)

> "You should actually consume the water as soon as you feel thirsty and ignore the food. Food costs water to process. Water is much more important than food. You can survive much longer without food. Dehydration quickly becomes debilitating and can kill you in a few days."

The logical flow is easy to follow, but it offers little beyond the chain of claims. It was labeled `reasoning` because the steps do connect cause to effect, even if the underlying facts read as common sense.

**3. `reasoning` vs. `assertion` → labeled `reasoning`** (hunger and thirst signals)

> "The feeling of hunger and thirst are more tied to your usual habits rather than actual need. Your body basically just gets used to your usual rhythm of water and food intake and sets a schedule of when to send signals to you to feel hungry and thirsty, and it does that via hormones."

The hardest of the three. The logic is coherent, but — as with case #2 — it was difficult to judge whether the comment genuinely explains the mechanism (hormonal scheduling) or simply asserts a "you should know this" claim while sidestepping the deeper *why*. It was labeled `reasoning` because it names an actual mechanism.

---

## Label Distribution

| Label | Count | Share |
|---|---:|---:|
| `analogy` | 69 | 34.5% |
| `assertion` | 55 | 27.5% |
| `reasoning` | 46 | 23.0% |
| `reaction` | 30 | 15.0% |
| **Total** | **200** | **100%** |

As anticipated in [planning.md](planning.md), the classes are imbalanced — `reaction` is the smallest at roughly 15%. This is expected when sampling *all* comments in a thread, and it directly shapes evaluation: overall accuracy alone would be misleading, so per-class precision/recall/F1 and a confusion matrix are used to keep the minority classes honest.
