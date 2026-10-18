# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy ≥ 0.60 on the held-out set

On the held-out test set, the trained model's overall accuracy is at
least 0.60.

**Why this target:** Four labels means a random guesser gets about
0.25. My largest label (`analysis`) is likely to cover around 40% of
the data, so a model that only ever predicts the majority class would
land near 0.40. 0.60 is a real improvement over both, and reachable
for a small fine-tuned model on 200 posts. I set it at 0.60 rather
than 0.65 because the test split will only have about 30 posts, and
accuracy on 30 examples moves by 0.03 for a single classification
change. 0.60 is a floor I can defend; 0.65 would be optimistic.

---

## 2. Macro F1 ≥ 0.55

The macro-averaged F1 across all four labels is at least 0.55 on the
held-out set.

**Why this target:** Macro F1 weights every label equally, so it
catches a model that learns the majority class and ignores the rest.
A model that hits 0.60 accuracy by only predicting `analysis` would
have a macro F1 near 0.20. 0.55 requires every label to be learned to
some degree, which is what a usable classifier needs.

---

## 3. Every label reaches F1 ≥ 0.40

No label scores below 0.40 F1 on the held-out set — including the
smallest one.

**Why this target:** A label below 0.40 is not being learned at all;
it is the model guessing or defaulting. I set 0.40 rather than 0.50
because my smallest label (`meme` or `reaction`) will only have about
8 examples in the test split, and an F1 on 8 examples moves a lot
between seeds. 0.40 is a real floor for 8 examples; 0.50 would set
the smallest label up to fail on variance alone.

---

## 4. No label exceeds 55% of the labeled set

In `labels.csv`, no single label accounts for more than 55% of the
rows.

**Why this target:** The brief's hard cap is 70%, but 70% is where a
model starts defaulting to the majority label. I set 55% so my labels
are balanced enough that a majority-class guesser scores around 0.55
at best, forcing the model to learn the distinctions. I could set 45%
as a target, but I can't control what the community actually posts —
55% is the balance I can plausibly hold while labeling.

---

## 5. Same seed produces the same accuracy to three decimals

Running the notebook twice with the same seed and the same data
produces the same accuracy to three decimal places.

**Why this target:** Reproducibility is binary — either the training
pipeline is deterministic or it isn't. If two runs with the same seed
disagree, there is a nondeterminism bug (a missing seed, a
non-deterministic data loader, a random augmentation) and no other
criterion can be trusted. Three decimal places is strict enough to
catch any real drift; looser would let a one-post flip through.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
