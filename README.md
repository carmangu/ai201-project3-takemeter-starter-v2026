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

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



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
