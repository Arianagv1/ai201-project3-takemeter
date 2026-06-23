# TakeMeter — Project Planning

A fine-tuned text classifier that evaluates discourse quality in an online community of my choosing. This document covers the label taxonomy, data collection strategy, and evaluation plan.

---

## Milestone One

### Community

> *What community did you choose and why? What makes its discourse varied enough to be interesting for a classification task?*

This project depends on an online community whose discourse is **active, text-heavy, and varied in quality** — and one large enough to supply the **minimum of 200 posts** the task requires.

For these reasons I chose the subreddit **r/ExplainLikeImFive (r/ELI5)**. It fits well because contributors are encouraged to explain concepts in their own words, drawing on factual data, analogies, and personal experience. Since these explanations vary widely in quality and credibility, the community offers the rich, natural variation needed to make the labels meaningful.

### Labels

> *What are your 2–4 labels? Define each in a complete sentence, with 2 example posts per label.*

TakeMeter classifies each comment by the **type of discourse** it represents, not by a subjective "quality" grade. Quality is never labeled directly — it *emerges* from the distribution of types, since a thread full of `reasoning` reads very differently from one full of `reaction`. The four types form an implicit quality gradient: `reasoning` and `analogy` are genuine explanations, `assertion` answers without explaining, and `reaction` adds no explanatory content at all.

(Clarity and completeness are deliberately **not** labels — they are too much "in the eye of the beholder" to annotate consistently. I track them only as informal annotation notes.)

The example posts below are drawn from one thread: *"ELI5 — Dividing by Zero. If I have 2 bananas and I divide them by nothing, how is the answer undefined and not just that I still have 2 bananas?"*

**Label 1 — `reasoning`**
The comment explains *why* its answer holds using a logical, factual, or mathematical argument.

- *Example post 1: Then there's the math explanation: Let's say 2/a = b. This means that a x b = 2. Now, if dividing by zero you have a = 0, but that means there is some number b where 0 x b = 2. That obviously doesn't work. @Encomiast*
- *Example post 2: The answer can't be "I still have two bananas" because you are one person. You didn't divide the bananas among zero people, you divided them among one person. If you divided them by zero people, there's nobody who has the bananas anymore. Where did the bananas go? Less conceptually: Division is "how many times can I subtract the divisor from the dividend?" and if the divisor is zero, then you repeatedly do 2-0-0-0... and keep going until you run out. Except you can do that an infinite number of times and still not be done. There's no answer that actually works, so the result is undefined. @Vorthod*

**Label 2 — `analogy`**
The comment explains *why* its answer holds through a comparison, everyday framing, or lived experience rather than a formal argument.

- *Example post 1: Division is the reverse of multiplication. If three bunches of five bananas adds up to fifteen, then fifteen divided into three bunches means those bunches have five bananas each. If you had 12 boxes, and none of them had any bananas in them, you'd have no bananas at all. How many empty boxes do you need before you get to two bananas in total? It's a trick question, you won't get to a total of 2 bananas with empty boxes. @krigr*
- *Example post 2: Forget the two banana thing. Let's make it one banana. If I divide the banana into 1 piece, it remains whole — 1 banana. If I divide a banana into zero pieces, it ceases to exist. @kiimosabe*

**Label 3 — `assertion`**
The comment states an answer but gives no mechanism — it tells you *that* something is true, not *why*. (Example: a flat "You just can't divide by zero, it's undefined.")

- *Example post 1: Yeah, if you divide 2 bananas by 0, then where do the bananas end up? You placed them into nonexistents?@Sideshowcomedy*
- *Example post 2: 
For me dividing is separating into however many groups.You can't separate 2 bananas into 0 groups of bananas.@Actual-Care*

**Label 4 — `reaction`**
The comment makes no explanatory attempt — it expresses appreciation, humor, or an emotional response. (Example: "lol this finally makes sense, thank you!")

- *Example post 1: This also makes it easy to understand why "infinity" is wrong answer. "Each person gets infinity of bananas" sounds completely bonkers. @amakai*
- *Example post 2: I'm 50 years old and this is the best answer to this question I've ever heard.@HelicopterUpbeat5199*

### Hard Edge Cases

> *What type of post will be genuinely ambiguous between two labels? How will you handle it during annotation, and why does it matter?*

The genuine ambiguity is between **`reasoning`** and **`analogy`**, because many ELI5 comments build a logical argument *and* wrap it in an everyday comparison. The divide-by-zero thread is full of these — nearly every answer reaches for bananas, piles, boxes, or people.

**Decision rule (primary explanatory device wins):** Strip the comparison away. If a logical argument still stands on its own, label it **`reasoning`**. If removing the comparison leaves nothing to explain the *why*, then the comparison **is** the explanation — label it **`analogy`**.

Worked example: a comment arguing "as you divide by smaller and smaller numbers the result grows arbitrarily large, so at zero it would be infinite" leans on a pile image, but the limit argument survives without it → **`reasoning`**. @krigr's empty-boxes comment collapses to nothing if you remove the boxes → **`analogy`**.

A second rule guards the noise floor: a comment containing *any* genuine explanatory attempt is never **`reaction`**. `reaction` is reserved strictly for comments with no explanatory content at all.

This matters because a borderline post is a signal that the taxonomy needs an explicit decision rule — pinning it down now keeps annotation of 200 examples consistent across the dataset.

### Data Collection Plan

> *Where will you collect examples? How many per label? What if a label is underrepresented after 200 examples?*

I will collect comments by browsing threads on r/ELI5, sampling **all comments — both top-level answers and nested replies** — so that low-effort `assertion` and `reaction` posts are represented, not just the polished explanations. This is what lets TakeMeter measure discourse quality across the *whole* conversation rather than only its best parts.

Because I'm sampling every comment, I expect the classes to be **imbalanced**: `reaction` and `assertion` may be common while `reasoning` is comparatively rare. I'll aim for a usable spread across the four types, and if a type remains underrepresented after 200 samples:

- **At ~5% or below**, the type is too sparse to learn from and the taxonomy needs revising — most likely by merging `reasoning` and `analogy` into a single `explanation` type (the project allows as few as two labels).
- **At ~10% or above** (roughly 20+ samples), the type is manageable and can be kept as is.

**— End of Milestone One —**

---

## Milestone Two *(to complete)*

### Evaluation Metrics

> *Which metrics will you use, and why are they right for this task? (Accuracy alone is not enough — explain what else you need and why.)*

*[to complete]*

### Definition of Success

> *What performance would make this classifier genuinely useful? What is "good enough" for deployment in a real community tool?*

*[to complete]*

### AI Tool Plan

**Label stress-testing.** Give the AI my label definitions and edge-case description, and ask it to generate 5–10 posts that sit on the boundary between two labels. If it produces posts I can't classify cleanly, my definitions need tightening — done *before* annotating 200 examples.

**Annotation assistance.** Decide whether to use an LLM to pre-label a batch of examples before reviewing them myself. If so, note which tool and how I'll track which examples were pre-labeled (for disclosure in the AI usage section).

**Failure analysis.** Give my list of wrong predictions to an AI tool and ask it to identify patterns before writing up the evaluation. Note what patterns I'll look for and how I'll verify them myself.
