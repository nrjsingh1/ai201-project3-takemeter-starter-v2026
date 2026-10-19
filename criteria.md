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

## 1. Macro F1 is at least 0.55 on the held-out test set, on all three seeds.

**Why this target:** My data is lopsided toward `opinion` (about 65% in my
Milestone 1 reading), so a model that always says `opinion` gets around 60%
accuracy but only about 0.25 macro F1. Macro F1 weights `experience` and
`evidence` equally with `opinion`, so 0.55 can only be reached if the model
actually gets the two small labels right some of the time, not just the big
one. I kept it at 0.55 rather than higher because with ~30 test posts and
three seeds, one bad seed is likely.

---

## 2. Every label (`experience`, `evidence`, `opinion`) has an F1 of at least 0.40 on the held-out test set, on all three seeds.

**Why this target:** With 200 posts and `opinion` near 60%, `experience` and
`evidence` will each have only about 6 posts in the test split, so one or two
misclassified posts move their F1 by 0.10 or more. 0.40 is the floor below
which I'd say the model hasn't learned that label at all. It is also the
check that the model isn't just learning "first person = `experience`" and
missing `evidence` entirely.

---

## 3. No single label makes up more than 70% of `labels.csv`.

**Why this target:** In my Milestone 1 reading `opinion` was already about
65% of claim-making comments, so without deliberately pulling `experience`
posts from Ask HN and vendor threads and `evidence` posts from technical
threads, my data would drift past 70%. Above that, the model can score well by
guessing `opinion` and criteria 1 and 2 stop meaning much. Checked by the
label shares in notebook section 2.

---

## 4. On the held-out test set, accuracy on the model's most-confident third of predictions is at least 10 percentage points higher than on its least-confident third, on all three seeds.

**Why this target:** My hardest boundary (`experience` vs. `evidence`, a
first-hand story that contains numbers) should be where the model is least
sure. If confidence carries any signal, those borderline posts should pile up
in the least-confident third and drag its accuracy down. 10 points is a
small gap because each third is only about 10 posts, so one post is worth 10
points. Checked by `accuracy_most_confident_third` and
`accuracy_least_confident_third` in `results.json`.

---

## 5. When I label the 30 staff posts under the staff taxonomy, I agree with the staff labels on at least 70% of them (21 of 30).

**Why this target:** This is the only outside check on whether I apply a
written rule the way its authors meant it, which is the same skill my 200
labels depend on. The staff taxonomy isn't mine and has a
`hot_take`/`analysis` "does the fact carry the argument" rule close to my own
`evidence`/`opinion` rule, so I expect to miss on the borderline posts. 70%
allows up to 9 disagreements out of 30. Checked by `agreement.py`.

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
