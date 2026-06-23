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
| Accuracy | 0.60 | **0.43** |
| Macro F1 | 0.59 | **0.31** |
| Weighted F1 | 0.60 | **0.36** |
| Unparseable responses | 0 / 30 | — |

The fine-tuned model **underperformed the zero-shot baseline by 17 points of accuracy**. The full analysis of why is in the [Evaluation Report](#evaluation-report) below — and that gap is the most informative result in this project.

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

---

## Evaluation Report

### What the committed output files show

- **`evaluation_results.json`** — the machine-readable record of the run: both models' accuracy on the same 30-comment test set, the improvement delta (`-0.1667`), test-set size, the label→id map, and the base model. It is the reproducible source of the headline numbers quoted here.
- **`confusion_matrix.png`** — a visual heatmap of the fine-tuned model's predictions against the true labels on the test set. It shows *where* the model's errors concentrate; the same matrix is transcribed as a table below so it reads cleanly in text.

### Overall result

| Metric | Baseline (zero-shot `llama-3.3-70b`) | Fine-tuned DistilBERT |
|---|---:|---:|
| Accuracy | 0.60 | 0.43 |
| Macro F1 | 0.59 | 0.31 |
| Weighted F1 | 0.60 | 0.36 |

Fine-tuning **lowered** every metric. This is a real, diagnosable failure mode — not a bug — and the rest of this report explains exactly what the model learned instead of the intended task.

### Per-class metrics, both models

**Baseline (zero-shot LLM):**

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `reasoning` | 1.00 | 0.43 | 0.60 | 7 |
| `analogy` | 0.53 | 0.73 | 0.62 | 11 |
| `assertion` | 0.62 | 0.62 | 0.62 | 8 |
| `reaction` | 0.50 | 0.50 | 0.50 | 4 |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `reasoning` | 0.40 | 0.57 | 0.47 | 7 |
| `analogy` | 0.42 | 0.73 | 0.53 | 11 |
| `assertion` | 1.00 | 0.13 | 0.22 | 8 |
| `reaction` | 0.00 | 0.00 | 0.00 | 4 |

The single most telling number: the baseline gave every class an F1 between 0.50 and 0.62 — it *attempted* all four. The fine-tuned model scored **0.00 on `reaction`** and **0.22 on `assertion`**: it effectively stopped predicting the two minority classes at all.

### Confusion matrix (fine-tuned model)

Rows are the **true** label; columns are what the model **predicted**. The diagonal (bold) is correct predictions.

| True ↓ \ Predicted → | reasoning | analogy | assertion | reaction | Total (actual) |
|---|---:|---:|---:|---:|---:|
| **reasoning** | **4** | 3 | 0 | 0 | 7 |
| **analogy** | 3 | **8** | 0 | 0 | 11 |
| **assertion** | 3 | 4 | **1** | 0 | 8 |
| **reaction** | 0 | 4 | 0 | **0** | 4 |
| **Total (predicted)** | 10 | 19 | 1 | 0 | 30 |

### Where the model fails — pattern analysis

An LLM (Claude) was used to help surface patterns across the errors; the findings below were then verified directly against the confusion matrix and cross-checked against all 17 misclassified texts (see the verified themes and worked examples below).

**The dominant pattern is class collapse, not a single confused pair.** Read the bottom row of totals: the model predicted `analogy` **19 of 30 times** and `reasoning` 10 times, but `assertion` only **once** and `reaction` **never**. It essentially became a two-class "explanation" classifier that defaults to `analogy`.

Three specific directional failures account for almost all the errors:

1. **`reaction` → `analogy` (4 of 4, 100%).** Every single `reaction` comment was labeled `analogy`. The model never learned the class exists.
2. **`assertion` → `reasoning`/`analogy` (7 of 8).** Bare claims were almost always read as explanations. Tellingly, `assertion` precision is 1.00 — *when* the model committed to `assertion` it was right, it just almost never committed.
3. **`reasoning` ↔ `analogy` (3 each way).** The boundary predicted as hard in [planning.md](planning.md) did show symmetric confusion — but it is now a *secondary* problem behind the minority-class collapse.

**Why the model failed — too little data for categories this subtle, not imbalance.** The dataset was reasonably balanced (`analogy` 34%, `assertion` 27%, `reasoning` 23%, `reaction` 15%), so imbalance is *not* the main cause. The real problem is that `reasoning`, `analogy`, and `assertion` **overlap heavily** — all three involve explaining or claiming, differing only in fine ways (*does it use a comparison? does it give a mechanism?*). Learning distinctions that subtle from ~140 examples (≈21–48 per class) is simply too little signal. The proof is in the confidence scores: **every one of the 17 wrong predictions landed at 0.28–0.29, and the 13 *correct* ones sit in the same 0.27–0.31 band** — all barely above the 0.25 floor for a blind 4-way guess. The model is no more confident when it is right than when it is wrong: it isn't *confidently* wrong, it is near-uniformly unsure on every comment, the signature of a model that never found discriminative features. Imbalance plays only a secondary role here: it decided *which* classes were abandoned first (the rarer, more distinct `reaction` and `assertion`), not *why* the model failed. Even a perfectly balanced 140 examples would likely struggle on a boundary this fine.

**Labeling problem or data problem?** Mostly a **data-volume problem**, not annotation inconsistency: the labels followed explicit, documented decision rules (see [Labeling Process](#labeling-process)), and `assertion`'s precision of 1.00 shows the model labels a clear assertion correctly *when* it commits — it just rarely does. But re-reading the errors surfaced one genuine **labeling/boundary issue** as well: my `analogy` definition includes "lived experience," which overlaps with `reaction` comments that are personal anecdotes (see example #1 below). So `reaction`↔`analogy` is partly a taxonomy-overlap problem — a point I had to *correct* from an initial "purely data" read.

**What would need to change.** In order of leverage: (1) **more minority-class data** — collect additional `reaction` and `assertion` comments to approach balance, since those are the easiest comment types to find; (2) **class-weighted loss**, so minority errors cost more during training; (3) **consider merging `reasoning`+`analogy` into a single `explanation` label** (the fallback noted in planning.md), which both removes the fuzziest boundary and doubles the examples per remaining class; (4) **a larger dataset overall** — 200 comments is small for a 4-way text classifier competing against a 70B-parameter baseline.

### Themes verified by re-reading the misclassified texts

After an LLM surfaced candidate patterns, I re-read all 17 wrong predictions to confirm or discard them:

- **Causal connectives trigger `reasoning`.** Bare assertions containing "because" or "which is why" were flipped to `reasoning` even though no mechanism follows (see example #2 below). The model learned a shallow surface shortcut — *connective ⇒ argument* — rather than checking whether the comment actually explains *why*.
- **First-person / lived-experience phrasing triggers `analogy`.** Personal asides were pulled into `analogy`, which connects directly to the label-overlap noted above.
- **Discarded patterns:** I found *no* evidence of sarcasm-driven errors, and post length did **not** separate right from wrong predictions (the model misfired on both short claims and long explanations). I left these out rather than force a pattern that wasn't there.

### Checkpoint instability (corroborating note)

Out of curiosity, I also evaluated the checkpoint selected on plain **accuracy** (the notebook default) instead of macro F1. It collapsed in the *opposite* direction: `analogy` recall fell from 0.73 to **0.09** — the model almost stopped predicting `analogy` and over-predicted `reasoning`/`assertion` instead — and test accuracy was *lower* at 0.367. That the entire prediction distribution flips between two checkpoints of the same model, **and** that the accuracy-selected one scored worse on the test set despite being chosen for high validation accuracy, confirms two things: the model has no stable class representation (a textbook underfitting signature), and the 30-example validation set is too small to reliably guide checkpoint selection.

### Three specific misclassifications

Three real test comments, one per dominant error type. Every prediction below carried ~0.29 confidence — the model was essentially guessing.

**1. `reaction` → `analogy`** (confidence 0.29)
> "My ex-wife had to be the driver or she would get sick. That was nice because I can read in the car without any issues, so I got a lot of reading done while on the road."

A personal aside that never explains the question (why motion sickness happens), so it is `reaction`. Two things converge: the model has *no* learned representation of `reaction` (it predicted that label 0 times), and the first-person, lived-experience phrasing surface-matches the `analogy` definition, which literally includes "lived experience." So this error is *both* data starvation **and** a real definitional overlap between `reaction` and `analogy`. **Fix:** more `reaction` data, plus a tighter rule that lived experience only counts as `analogy` when it is used to *explain the question*.

**2. `assertion` → `reasoning`** (confidence 0.29)
> "Because organic chemistry is incredibly important economically and biologically."

This states *that* carbon's chemistry matters, not *why* it gets its own branch — a bare claim, so `assertion`. The likely trigger is the opening word **"Because"**: the model learned a shallow shortcut where a causal connective signals an argument, regardless of whether a mechanism follows. The same flip appears in the water-rationing error ("…*which is why* it is important not to ration water"). **Fix:** more `assertion` examples that contain "because / so / which is why" *without* a real mechanism, so the model stops keying on the connective.

**3. `analogy` → `reasoning`** (confidence 0.28)
> "Hot air and cold air both want to go down, but cold air is denser than hot air and when it goes down it says 'out of my way, hot air!'"

This personifies air (*"it says 'out of my way, hot air!'"*) — a textbook everyday-framing `analogy`. But it also contains a real causal clause ("cold air is denser… when it goes down"), placing it exactly on the planning.md hard boundary: argument *and* comparison in one comment. The "primary explanatory device" rule resolves it for a human, but the model can't detect the comparison device and defaults to the structural argument. (The same pattern appears in the mushroom-cloud and pilots/ship-captains errors.) **Fix:** more mixed argument-plus-comparison examples, or merge the two labels into `explanation`.

### Sample Classifications

Test comments run through the fine-tuned model, shown with predicted label, confidence, and true label. The headline behavior is visible at a glance: **confidence never leaves the ~0.27–0.31 band whether the prediction is right or wrong** — just above the 0.25 chance floor. A model that is no more confident when correct than when wrong is essentially guessing.

| Comment (abbreviated) | Predicted | Confidence | True | Correct? |
|---|---|---:|---|:--:|
| "Firefighters are trained in moving unconscious people around. When my mother in law had what we thought was a stroke…" | `analogy` | 0.28 | `analogy` | ✅ |
| "My ex-wife had to be the driver or she would get sick… I got a lot of reading done…" | `analogy` | 0.29 | `reaction` | ❌ |
| "Because organic chemistry is incredibly important economically and biologically." | `reasoning` | 0.29 | `assertion` | ❌ |
| "Hot air and cold air both want to go down… it says 'out of my way, hot air!'" | `reasoning` | 0.28 | `analogy` | ❌ |
| "your brain's basically running constant physics simulations from past experience" | `analogy` | 0.28 | `assertion` | ❌ |

**Why the correct prediction is reasonable:** the firefighters comment explains *why* fire engines accompany ambulances entirely through a personal anecdote (helping move an unconscious relative) rather than a formal argument — lived experience used as the explanatory device — so `analogy` is genuinely the right call. Tellingly, its confidence (0.28) is no higher than the model's *wrong* predictions, so even this hit reads more like a lucky landing than a confident decision.

### Reflection: what the model captured vs. what I intended

I intended a **four-way discourse-type classifier** that could tell genuine explanation (`reasoning`, `analogy`) apart from bare claims (`assertion`) and non-contribution (`reaction`) — the full quality gradient that makes TakeMeter a *discourse-quality* meter.

What the model actually learned is much narrower: **a near-binary "does this contain explanation-like content?" detector that leans toward `analogy`.** Its decision boundary captures roughly one axis — *amount of content* — rather than the four *types* I defined. And it learned even that weakly: with confidence pinned near 0.29 on every comment, it never formed strong preferences at all. The failure is better described as **underfitting than overfitting** — 140 examples across four overlapping categories was too little to learn any of the fine distinctions, so the model fell back on the broadest, safest guesses.

What it missed is precisely the part that gives the taxonomy its point. `reaction` (recall 0.00) and `assertion` (recall 0.13) — the low end of the quality gradient — are exactly the classes the model erased. They were both the rarest *and* the subtlest to separate from an explanation, and that combination is fatal: the distinctions hardest to learn from few examples are the very ones that separate *noise* from *substance*, which is the whole reason the tool exists.

The sharpest framing of the gap is the comparison itself: the zero-shot LLM gave all four classes a real shot (F1 0.50–0.62) because its pretraining already contains the *concepts* of a reaction or a bare assertion — it needs no examples. My fine-tuned model had to learn those concepts from scratch, and 200 labeled comments were not enough to teach them. **The taxonomy was sound; the dataset was too small — and the four categories too subtly different — to instill it.** That is the central, diagnosable lesson of this build, and it points to a concrete, testable fix: more data and/or fewer, better-separated labels (e.g., merging `reasoning` and `analogy` into `explanation`) before any further tuning.

---

## Spec Reflection

**How the spec guided the build.** The decision rules I committed to in [planning.md](planning.md) — "primary explanatory device wins" and the `reaction` noise floor — did real work twice. First, they kept annotation consistent across 200 comments by giving me a rule to fall back on whenever a comment sat between two labels. Second, I copied the label definitions *and* those rules verbatim into the zero-shot baseline prompt, so the LLM judged comments by the same criteria I annotated by. That alignment is the only reason the baseline-vs-fine-tuned comparison is meaningful — both systems were measured against the same definition of each label.

**How the implementation diverged, and why.** The spec anticipated exactly one hard boundary — `reasoning` vs `analogy` — and wrote a single decision rule for it. During annotation a *second*, unplanned ambiguity surfaced: personal anecdotes that sit between `reaction` and `analogy`, because my `analogy` definition explicitly includes "lived experience" while many `reaction` comments are personal asides. The spec didn't foresee this, so I made case-by-case calls during annotation, and the evaluation later confirmed it as a genuine taxonomy overlap (the firefighters and ex-wife examples). The divergence happened because real, varied data exposed a boundary the planning phase couldn't predict from the single divide-by-zero thread used for the original examples — itself a finding worth carrying into a v2 of the label set.

## AI Usage

AI assistance (Claude) was used during planning, prompt-writing, and analysis — not during annotation. Specific instances:

**1. Label taxonomy design.** I directed Claude to critique my initial labels (Simplicity, Completeness, Cites Reasoning, Use of Analogy). It identified that these mixed two incompatible axes — *quality* (simplicity, completeness) and *method* (reasoning, analogy) — which cannot be ranked on a single scale, and recommended reframing around discourse *types*. It produced the `reasoning / analogy / assertion / reaction` scheme. **What I decided/overrode:** I chose the community (r/ELI5) and the single-label-with-edge-case-override structure myself, and made the call to sample all comments including nested replies — Claude advised on these but did not make the decisions.

**2. Baseline classification prompt.** I directed Claude to write the zero-shot Groq prompt from my planning.md definitions, formatted for clean single-label output. It produced a four-label prompt with one example per label plus the two decision rules. **What I changed:** I verified the label strings matched my CSV exactly before running, since any mismatch would have marked every response unparseable.

**3. Failure-pattern analysis (the assignment's AI step).** I pasted my misclassified test examples into Claude and asked it to surface common themes. It proposed a causal-connective shortcut ("because" → `reasoning`), a lived-experience → `analogy` overlap, and near-chance confidence across all predictions. **What I overrode:** Claude's first explanation leaned on *class imbalance* as the main cause; I pushed back because my dataset was roughly balanced (~15–34% per class), and the analysis was corrected to attribute the failure to **data volume plus subtly overlapping labels**, with imbalance demoted to a secondary factor. I verified each surfaced pattern by re-reading the test examples and confirmed the discarded ones (no sarcasm effect; length did not separate errors).

**Annotation disclosure.** All 200 comments were labeled manually by me. I did **not** use an LLM to pre-label any part of the dataset; AI assistance was limited to label design, prompt-writing, and post-hoc failure analysis as described above.
