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
| `reasoning` | 47 | 23.0% |
| `reaction` | 30 | 15.0% |
| **Total** | **200** | **100%** |

As anticipated in [planning.md](planning.md), the classes are imbalanced — `reaction` is the smallest at roughly 15%. This is expected when sampling *all* comments in a thread, and it directly shapes evaluation: overall accuracy alone would be misleading, so per-class precision/recall/F1 and a confusion matrix are used to keep the minority classes honest.

---

## Baseline Comparison

Before fine-tuning, a **zero-shot baseline** was established to measure how hard the task is for a general-purpose model with no training. This gives the fine-tuned model's score real meaning: beating a no-training LLM is the minimum bar for fine-tuning to have been worthwhile.

### Baseline approach

The baseline uses **`llama-3.3-70b-versatile`** (via the Groq API) as a zero-shot classifier — no examples from the training data, no fine-tuning. A single system prompt:

1. names the community (r/ELI5) and the classification task,
2. defines all four labels in plain language (copied from [planning.md](planning.md)),
3. includes one real example comment per label drawn from the dataset, and
4. instructs the model to output **only** the lowercase label name, so the notebook's parser can match it exactly.

The model classified each of the **30 comments in the locked test set** (15% of the 200-comment dataset, held out and never used for training). Every response was parsed against the exact label strings. **The fine-tuned model is evaluated on this same 30-comment test set**, so the two systems are directly comparable.

### Results on the shared test set

| Metric | Baseline (zero-shot `llama-3.3-70b`) | Fine-tuned DistilBERT |
|---|---:|---:|
| Accuracy | 0.60 | _TBD_ |
| Macro F1 | 0.59 | _TBD_ |
| Weighted F1 | 0.60 | _TBD_ |
| Unparseable responses | 0 / 30 | — |

Per-class metrics for the baseline:

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `reasoning` | 1.00 | 0.43 | 0.60 | 7 |
| `analogy` | 0.53 | 0.73 | 0.62 | 11 |
| `assertion` | 0.62 | 0.62 | 0.62 | 8 |
| `reaction` | 0.50 | 0.50 | 0.50 | 4 |

### What the baseline reveals

The headline number — 60% accuracy against a ~25% random-chance floor for four classes — shows a general model can do *something* with the task but is far from reliable. The per-class split is more telling:

- **`reasoning` has perfect precision (1.00) but low recall (0.43).** The model only labels a comment `reasoning` when it is certain, catching just 3 of 7 — the other 4 slip through.
- **`analogy` is the catch-all** (precision 0.53, recall 0.73): it absorbs comments the model is unsure about, which is almost certainly where the missed `reasoning` comments landed.

This confirms the hypothesis from [planning.md](planning.md): **`reasoning` and `analogy` are the genuinely hard boundary.** `assertion` is mediocre but unbiased, and `reaction` (support 4) is too small to draw firm conclusions from.

**Hypothesis to test after fine-tuning:** fine-tuning should chiefly (1) raise `reasoning` recall by recovering comments currently misfiled as `analogy`, and (2) tighten `analogy` precision as a result. This will be checked against the fine-tuned model's confusion matrix.

---

## Model & Fine-Tuning

TakeMeter fine-tunes **`distilbert-base-uncased`** — a compact (~67M-parameter) language model — by attaching a fresh 4-way classification head and continuing training on the 70% training split (~140 comments). Training ran on a Colab T4 GPU.

### Training configuration

| Setting | Value |
|---|---|
| Epochs | 3 |
| Learning rate | 2e-5 |
| Batch size | 16 |
| Weight decay | 0.01 |
| Warmup steps | 50 |
| Best-checkpoint metric | macro F1 (changed from default `accuracy`) |

### Key training decision: selecting on macro F1, not accuracy

The one default I changed was the metric used to pick the best checkpoint: from **accuracy** to **macro F1**. With `load_best_model_at_end=True`, the trainer keeps whichever epoch scores highest on the validation set — so this choice directly determines which model ships.

Accuracy is dominated by the majority class (`analogy`): a checkpoint could excel on `analogy` and fail the minority `reaction` class while still posting a high accuracy score. Macro F1 averages the per-class F1 scores **equally**, so weakness on any single label — including the small ones — is penalized just as heavily. Switching to it keeps the *training* objective consistent with this project's imbalance-aware evaluation, where overall accuracy alone is treated as misleading.

### Key training decision: holding epochs at 3

The most consequential knob on a dataset this small is **how many epochs** to train, and I deliberately kept it at **3** rather than increasing it — for a concrete reason, not just because it was the default.

With only ~140 training examples, more epochs would let the model start **memorizing** individual comments (overfitting) instead of learning the general signal that separates the four discourse types. The risk is highest exactly here: a model with 67M parameters can trivially memorize 140 short texts if given enough passes.

Just as decisive is the **measurement problem**: the test set is only 30 comments, so a single example is worth ~3.3% accuracy. Any improvement from extra epochs would likely be *smaller than the noise* of that tiny test set — meaning I couldn't reliably tell whether a change helped or just got lucky. Tuning that can't be measured isn't worth the overfitting risk, so the conservative defaults (3 epochs, learning rate 2e-5, weight decay 0.01) are the principled choice for this data regime.
