# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter classifies posts from r/nba into four labels: `analysis`,
`hot_take`, `reaction`, and `meme`. The goal is to sort high-effort
discussion from emotional noise and community in-jokes. The classifier
is a fine-tuned small language model, trained on 200 posts I labeled
by hand from the public subreddit.

---

## Label Taxonomy

Community: r/nba. Posts are sorted into four labels.

### `analysis`

**Definition:**
Explains why something happened, using checkable
evidence (stats, tactics, history) and proposing a mechanism — not
just a conclusion.

**Example 1:**
> The Knicks' offensive rebound rate this series is 38%, driven mostly
> by Hart timing his weak-side cuts.

**Example 2:**
> Comparing the 2022 and 2026 defenses: why drop coverage stopped
> working against Luka.

### `hot_take`

**Definition:**
A confident subjective claim about rankings, legacy, or
trades, made to provoke argument rather than explain anything.

**Example 1:**
> If Wemby wins this year, is he already above KD all-time?

**Example 2:**
> Thibs' minutes rotation will never win a championship.

### `reaction`

**Definition:**
An immediate emotional response to a specific play or
moment, with no argument attached. Usually short, usually about the
last play or the current score.

**Example 1:**
> OH MY GOD BRUNSON

**Example 2:**
> What is this defense? I'm going to bed.

### `meme`

**Definition:**
A post whose primary purpose is humor, satire, or a
running community joke, not a serious take.

**Example 1:**
> Give the WCF MVP to the refs. [image: ref in a Knicks jersey]

**Example 2:**
> Ty Lue said he's working on making a 20-point lead look like a
> 20-point deficit.

### The hardest boundary

**Which two labels:** `analysis` and `hot_take`.

**The decision rule I used every time:** If the post proposes a
*mechanism* — a chain of cause and effect a reader could check
("because the weak-side rotation was late, the corner was open") — it
is `analysis`, even if the tone is heated. If the post only stacks
evidence to support a preset verdict and never decomposes the
mechanism, it is `hot_take`.

Borderline example that defined the rule:

> Brunson is shooting 38% from the field this series. People keep
> calling him a top-5 PG but the numbers say he's been average. If the
> Knicks lose this round, he's not a top-10 player.

This has a stat (38%) and a verdict (not top-10). It reads like
analysis. But it never explains *why* the number is low — no
defensive scheme, no shot-quality breakdown. The stat is decoration
for a preset conclusion, so it is `hot_take`.

---

## The Dataset

**Where the posts came from:** Public text posts from r/nba, collected via the Arctic Shift Reddit archive API. I only kept posts with sufficient text (filtered out highlights, videos, and pure links) to ensure the model had actual content to classify.

**How I labelled them:** I manually labeled the first 20 posts without any AI help to establish a baseline (`cold`). For the remaining 160 posts, I used an AI tool to pre-label batches based on my definitions, then manually reviewed and corrected every single one (`pre-labeled`).

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `reaction` | 114 | 63.3% |
| `analysis` | 46 | 25.6% |
| `hot_take` | 18 | 10.0% |
| `meme` | 2 | 1.1% |
| **Total** | 180 | 100% |

**Three hard cases**

**1.**
> *The post:* "Suns owner Matt Ishbia on trading for Kevin Durant... Kevin durant then claps back on Twitter..."
> *Could have been:* `hot_take` / `reaction`
> *I chose `reaction`, because:* It's a direct transcript of a social media beef. Although it involves strong opinions, it's primarily reporting immediate emotional responses rather than a standalone argument.

**2.**
> *The post:* "A decade ago most of this sub was white people. Now it seems there a lot of asians/Indians on here. The most obvious sign is how much engagement Jeremy Lin threads get..."
> *Could have been:* `hot_take` / `analysis`
> *I chose `hot_take`, because:* It makes a broad sociological claim without providing checkable evidence or a causal mechanism, aiming mostly to provoke discussion.

**3.**
> *The post:* "The ""face of the league"" is more of who can gain the most following around their image... Jokic got close since his game is fun to watch..."
> *Could have been:* `analysis` / `hot_take`
> *I chose `hot_take`, because:* While long and detailed, it relies on subjective criteria (coolness, jerseys, personality) rather than objective, checkable metrics or tactical mechanisms.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Max sequence length: 128
- Device: CPU
- Torch version: 2.14.1+cpu
- Seed: 42

**Anything I changed from the defaults, and why:** Nothing. I used the default settings.

**Split sizes:**

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 32 | 7 | 7 |
| `hot_take` | 12 | 3 | 3 |
| `reaction` | 80 | 17 | 17 |
| `meme` | 2 | 0 | **0** |
| **Total** | **126** | **27** | **27** |

**Note on `meme`:** With 2 examples total and 0 in the test split, `meme` cannot be evaluated on this run. Its F1 is 0.000 by definition, and no amount of training fixes that without more data.

**Training result (seed 42):**

```
accuracy  0.630
macro F1  0.258

analysis   F1 0.000
hot_take   F1 0.000
reaction   F1 0.773
meme       F1 0.000
```

The model predicted `reaction` on every test example, which gives 63% accuracy because `reaction` is 63% of the data. It learned the majority class and nothing else. That is a real finding to diagnose in the next unit, not a mistake in this unit's setup.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked Claude to help me tighten my label definitions for `analysis` vs `hot_take`.
- *What came back:* Its first suggestion was "analysis uses data, hot_take does not." That is too loose — many hot takes cite stats.
- *What I changed:* I rewrote the decision rule around *mechanism* instead of *evidence*: analysis proposes a chain of cause and effect a reader could check; hot_take uses evidence only to support a preset verdict. That is what the rule in my `Label Taxonomy` section says.

**Moment 2**

- *What I asked for:* I gave Claude my label definitions and batches of 30 posts at a time, asking it to label each post.
- *What came back:* Its labels for `reaction` and `hot_take` were good, but it over-labeled `analysis` for posts that used stats without proposing a mechanism. That is exactly the boundary I had written a rule for, applied the wrong way.
- *What I changed:* I manually re-checked every AI-labeled row against my decision rule and corrected the `analysis` → `hot_take` errors. Every row in `labels.csv` marked `pre-labeled` was reviewed by me after AI pre-labeling; the first 20 rows are marked `cold` because I labeled them with no help.

**Pre-labelling disclosure:**

Yes. The first 20 rows were labeled by hand without Claude. The remaining 160 were Claude pre-labeled and then manually reviewed and corrected. `meme` has only 2 examples total — that is not a pre-labeling error, it is a genuine scarcity in the data.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.296 | 0.630 | +0.333 |
| Macro F1 | 0.134 | 0.258 | +0.124 |
| F1 — `analysis` | 0.424 | 0.000 | -0.424 |
| F1 — `hot_take` | 0.000 | 0.000 | +0.000 |
| F1 — `meme` | 0.000 | 0.000 | +0.000 |
| F1 — `reaction` | 0.111 | 0.773 | +0.662 |

**What I predicted before I looked:**
I predicted that the trained model would beat the baseline on overall accuracy, because the baseline is a zero-shot general model that doesn't understand the r/nba context. However, given that my trained model previously collapsed and predicted `reaction` for everything, I expected the gap to be relatively small, with both models struggling on minority labels like `hot_take` and `meme`.

**What the gap actually means:**
The trained model's higher accuracy (0.630) is entirely driven by learning the majority class (`reaction`), which makes up 63% of the data. The baseline has a *higher* F1 on `analysis` (0.424 vs 0.000), meaning the fine-tuned model actively forgot how to classify `analysis` in order to maximize accuracy on `reaction`. This is a classic case of majority-class collapse, confirming that the fine-tuning process learned the class distribution but not the underlying boundaries between labels.

---

## Run Log — Before

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy ≥ 0.60 | 0.60 | 0.630 | 0.630 | 0.630 | MET |
| 2. Macro F1 ≥ 0.55 | 0.55 | 0.258 | 0.258 | 0.258 | MISSED |
| 3. Every label reaches F1 ≥ 0.40 | 0.40 | 0.258 | 0.258 | 0.258 | MISSED |
| 4. No label exceeds 55% of the labeled set | 55% | 63.3% | 63.3% | 63.3% | MISSED |
| 5. Same seed produces the same accuracy | = 0.630 | 0.630 | 0.630 | 0.630 | MET |

**Note on seed stability:** The accuracy and Macro F1 were identical (0.000 spread) across all three seeds. This confirms that the training pipeline is deterministic and that the model's collapse (predicting `reaction` for every post) is not a random fluke but a stable behavior caused by the class imbalance.

### Confusion matrix

| true \ predicted | analysis | hot_take | reaction | meme |
|---|---|---|---|---|
| **analysis** | 0 | 0 | 7 | 0 |
| **hot_take** | 0 | 0 | 3 | 0 |
| **reaction** | 0 | 0 | 17 | 0 |
| **meme** | 0 | 0 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
7 real `analysis` posts were called `reaction`. The model is not confusing one boundary for another; it has completely abandoned all labels except `reaction`. The dominant error direction is strictly one-way (all non-reaction posts go to reaction), which confirms the model's total majority-class collapse.

---

## Verdicts and Diagnoses

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy ≥ 0.60 | 0.60 | **MET** | 0.630 in all three seeds, barely above the target. |
| 2 | Macro F1 ≥ 0.55 | 0.55 | **MISSED** | 0.258 across all seeds, far below 0.55. |
| 3 | Every label reaches F1 ≥ 0.40 | 0.40 | **MISSED** | Only `reaction` scored above 0.40; `analysis`, `hot_take`, `meme` all scored 0.000. |
| 4 | No label exceeds 55% | 55% | **MISSED** | `reaction` is 63.3% of the labeled set, exceeding the target and the hard cap of 70%. |
| 5 | Same seed accuracy | = 0.630 | **MET** | Seed 42 produced the exact same accuracy (0.630) across multiple runs, confirming deterministic training. |

**Diagnoses**

My agreement report reveals a clear pattern: I applied the staff's written rules too loosely, consistently misclassifying posts that had a negative or dismissive tone. This same inconsistency likely infected my own `r/nba` dataset.

- **Criterion 2 (Macro F1 MISSED):** The model collapsed and predicted `reaction` for every test example. This is a *labeling* problem, not just a *model* problem. My agreement check proves I applied boundaries inconsistently, so my training data for `analysis` and `hot_take` is likely noisy. The model learned the dominant signal (`reaction` = majority) because the minority class boundaries were unclear.
- **Criterion 3 (Every label F1 MISSED):** Same as above. The model did not learn meaningful boundaries for `analysis`, `hot_take`, or `meme` because I failed to label them with high consistency. The 0.000 F1 for `analysis` and `hot_take` is a direct symptom of my own inconsistent labeling, not just a lack of data.
- **Criterion 4 (Balance MISSED):** This is a dataset collection problem. I have 114 `reaction` posts (63.3%) and only 2 `meme` posts. The imbalance forced the model into a majority-class collapse. I would need to collect more `meme` and `hot_take` posts to fix this.

---

## Agreement Report

**Agreement rate:** 28 / 30 = 93%

**Disagreements**

**1.**
> *The post:* "Carbon plates aren't the whole story. My times dropped 2% in the shoes and another 2% the following year in the same pair, when the only thing that changed was going from 35 to 50 miles a week."
>
> *Staff said / I said:* Staff said `analysis` / I said `hot_take`
>
> *My call, and why:* Staff is right, and I applied the rule loosely. Rule 1 states that a fact must carry the argument. The author here uses two specific numbers (2% and 2%) to actively dismantle the claim that "shoes are the whole story." The fact does the argumentative work. I mistakenly downgraded it to `hot_take` because the writer's overall tone is dismissive, but under the staff's rule, the checkable facts are load-bearing.
>
> *Which pile:* (a) the rule covered it and I applied it loosely — a consistency problem.

**2.**
> *The post:* "Third DNF this year. I don't really want to talk about it, just wanted to put it somewhere."
>
> *Staff said / I said:* Staff said `reaction` / I said `hot_take`
>
> *My call, and why:* Staff is right, and this is a limitation of my own definition reading. Rule 2 states that a `reaction` reports what just happened to the writer and how it felt. The phrase "this year" plus "wanted to put it somewhere" shows an immediate, personal emotional response to a recent event (a DNF). I classified it as `hot_take` because "Third DNF" feels like a summary statement, but the staff taxonomy includes this as an immediate emotional disclosure, not a general claim about the world.
>
> *Which pile:* (a) the rule covered it and I applied it loosely — a consistency problem.

**What the pattern in my disagreements tells me:**
Both disagreements fall into pile (a) — I applied the staff's written rules too loosely. The pattern is that I tend to over-classify things as `hot_take` when the tone is negative or dismissive, ignoring the underlying structural features of the text. Rule 1 (load-bearing facts) and Rule 2 (personal immediate experience) both point to the staff being correct. This suggests that when I labeled my own `r/nba` dataset, I likely made similar errors on negative-sounding posts, treating facts as decoration or personal reactions as opinions. This is a significant, actionable diagnosis for my model's failure.


---

## The Improvement

**What I changed:** I added 30 new `hot_take` posts to the training data, increasing the total from 180 to 210. This reduced `reaction`'s share from 63.3% to 54.3% and brought `hot_take` from 10.0% to 22.9%, making the dataset more balanced.

**Which diagnosis pointed at it:** My diagnosis for Criterion 4 (MISSED) showed that `reaction` at 63.3% was the majority class, causing the model to collapse into predicting it for every test example. By increasing `hot_take` examples, I aimed to force the model to learn a second class.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy ≥ 0.60 | 0.60 | 0.562 | 0.562 | 0.562 | **MISSED** |
| 2. Macro F1 ≥ 0.55 | 0.55 | 0.240 | 0.240 | 0.240 | **MISSED** |
| 3. Every label reaches F1 ≥ 0.40 | 0.40 | 0.240 | 0.240 | 0.240 | **MISSED** |
| 4. No label exceeds 55% of the labeled set | 55% | 54.3% | 54.3% | 54.3% | **MET** |
| 5. Same seed produces the same accuracy | = 0.562 | 0.562 | 0.562 | 0.562 | **MET** |

**Did it help, and how do I know:**
The improvement helped with **Criterion 4 (Balance)** — the label share dropped from 63.3% to 54.3%, bringing it below the 55% target. It also moved the model away from pure majority-class collapse.

However, it made the other criteria slightly worse: overall accuracy dropped from 0.630 to 0.562, and macro F1 from 0.258 to 0.240. This is because the dataset is now more balanced, so the "easy" 63% accuracy from guessing `reaction` is no longer available. The model still fails to learn `analysis` and `hot_take` entirely (both F1 = 0.000).

This is a genuine finding: **adding data to the minority class helps balance the dataset but doesn't solve the underlying problem that the model cannot distinguish between the three labels from only ~30 examples per class in the test split.**

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
