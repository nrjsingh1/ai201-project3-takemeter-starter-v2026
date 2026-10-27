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

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter sorts top-level [Hacker News](https://news.ycombinator.com)
comments by what their claim rests on. `experience` comments argue from
something that happened to the writer, `evidence` comments argue from a fact
a reader could check, and `opinion` comments state a view with no support.
It is a fine-tuned DistilBERT trained on 293 labelled comments from 14
threads. On its first run (seed 42) it learned to call everything `opinion`;
diagnosing that is unit 6's work.

**Community:** [Hacker News](https://news.ycombinator.com) comment threads.
Public, no login needed to read, text-only, and spread across many topics
(tech, business, science, programming languages), so no single show or release
dominates. Comments range from one-line quips to long first-hand reports.

**Milestone 1 reading notes.** I read 36 top-level comments across four
front-page threads (a registrar seizing a user's domains, Terry Tao on AI in
mathematics, Java sealed classes, and an essay on "personal software
factories"). Candidate distinctions I saw in the text:

1. **Some report something that happened to the writer; most state a general
   verdict.** "Fully turned off our business domain because we didn't reply to
   a phishing email in 48 hours" vs. "Namecheap cannot be trusted."
2. **Some cite a specific, checkable fact (a number, date, link, or named
   event); most don't.** "purchased by private equity CVC in 2025 for $1.5B"
   vs. "modern math is not that important and mostly ego driven."
3. **Some are a one-line quip or verdict with no reason given.** "Scala
   punching the air rn", "Not very convincing", "There's a reason it's called
   Namecheap not Namesafe."
4. **Some are genuine questions asking for information rather than making a
   claim.** "What are good alternatives to namecheap?", "What's a software
   factory?"

Possible fifth, seen twice: some attack the motive of the person instead of
the argument ("I find this entirely self serving").


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

Three labels. The question behind all three: **what is this comment's claim
resting on?** The writer's own first-hand events, a fact anyone could check,
or nothing.

### `experience`

**Definition:** Describes a specific thing that happened to the writer or
their team (something they did and what resulted) and uses it as the basis
for what they say.

**Example 1:**
> Yea they did something like this with us. Fully turned off our business
> domain because we didn't reply to a phishing email in 48 hours. We migrated
> out the next day once we got it resolved. It took over 6 hours from when we
> noticed the issue and responded until they turned it back on. Absolutely
> horrible service.

**Example 2:**
> Namecheap cannot be trusted. I purchased a domain for a "normal" price (and
> yes, I have receipts). Shortly after they realized they fucked up, voided
> the purchase, and made it a $1,500 premium domain. Obviously, my
> recommendation is to avoid Namecheap.

### `evidence`

**Definition:** Makes a claim supported by a specific fact a reader could
check without trusting the writer: a number, a date, a quote, a named public
event, or a link whose content the comment states.

**Example 1:**
> namecheap purchased by private equity CVC in 2025 for $1.5B so costs need to
> go down (-human support + ai automation) and prices go up (they increased i
> think .com 3 times this year alone)

**Example 2:**
> Java also has recursive patterns and (maybe still in preview) literal
> patterns: `switch (x) { case A(B(true, var c)) -> f(c); }`

### `opinion`

**Definition:** States a view, verdict, or feeling with no support offered,
including one-line quips. It might be true; it isn't backed.

**Example 1:**
> For me it just shows that modern math is not that important and mostly ego
> driven. If it was important, mathematicians would cherish the progress. I
> can't imagine bio researchers get mad that AI cured cancer.

**Example 2:**
> Everything is being rethought right now. So many paradigms are broken.
> There wasn't ever a better time to build a software factory.

### What doesn't get a label

Pure questions ("What are good alternatives to namecheap?"), bare links with
no text, and deleted or non-English comments are skipped at collection. They
make no claim, so none of the three labels applies. In my Milestone 1 reading
that was about 5 of 36 comments.

### The hardest boundary

**Which two labels:** `experience` vs. `evidence`

The post that sits between them, from the Namecheap thread:
> I've found really great but quite unique .com domain names available on name
> cheap then mysteriously gone registered to name cheap within hours,
> available for purchase at thousands. Always strenuously denied by the name
> cheap CEO "we do not front run domain names!". I'd never do a domain name
> check there again.

It has specifics ("within hours", "at thousands") and a public statement by
the CEO, but the core claim rests on what happened to the writer.

**The decision rule I used every time:**
Ask where the support comes from. If the only way to verify it is to trust
the writer's account of their own events ("I", "we", "my team"), it's
`experience`, **even if it contains numbers**. If a reader could check it
somewhere else (a public price, a release note, a news event, a quoted
document), it's `evidence`. If a comment has both, label it by which kind the
conclusion rests on; if that is still a tie, pick whichever the comment spends
more words on. The post above is `experience`: the CEO quote is context, and
the claim ("they front-run domains") rests on the writer's own searches.

**Two smaller rules for the `opinion` boundaries** (both came out of the
stress test below):

- *`evidence` vs. `opinion`:* the fact has to be **stated and doing work**.
  Naming a source without saying what it shows ("the survey proves it") or
  tacking a number onto a conclusion that doesn't depend on it is `opinion`.
- *`experience` vs. `opinion`:* there has to be a **specific event with an
  outcome beyond the writer's feelings**. "I tried Kotlin and hated it" and
  "everyone I know who used GoDaddy got burned" are `opinion`.

### Stress test

I had an AI write eight comments that sit on these boundaries, then
classified each one under my draft definitions.

| # | Comment | Label | Clean? |
|---|---|---|---|
| 1 | "We moved 40 domains off Namecheap last year; renewals were up 22% across the board." | `experience` | Yes, via the main rule |
| 2 | "Namecheap's .com renewal went from $10.98 to $15.98 per their own pricing page. They're squeezing customers." | `evidence` | Yes |
| 3 | "Rust is the future, the 2025 Stack Overflow survey proves it." | `opinion` | **No.** Draft didn't say the fact had to be stated → added "stated and doing work" |
| 4 | "I tried Kotlin for a week and hated it. Java is just better." | `opinion` | **No.** Draft `experience` was "first-hand account" → added "outcome beyond feelings" |
| 5 | "This gave me the final kick to move away from Namecheap." (real) | `opinion` | **No.** Same fix as #4 |
| 6 | "Here's the CVE: [link]. This is exactly why you don't roll your own crypto." | `evidence` | Yes, the comment says what the link is |
| 7 | "At my last job our Postgres cluster did 50k writes/sec, so the 'web scale' pitch never impressed me." | `experience` | Yes, via the main rule |
| 8 | "Everyone I know who used GoDaddy got burned." | `opinion` | **No.** Generalized, no specific event → covered by the #4 fix |

### Expected imbalance

Of the ~31 claim-making comments I read in Milestone 1, about 20 were
`opinion`, 5 `experience`, and 5 `evidence`. That puts `opinion` near 65%,
close to the 70% cap. Collection will need to target threads that draw
first-hand accounts (Ask HN, outage and vendor post-mortems) and technical
threads that draw facts.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Top-level comments from 14 Hacker News
threads, pulled through the public HN Algolia API (`hn.algolia.com/api/v1/items/<id>`)
in October 2026. I picked threads to balance the labels: vendor complaints
and Ask HN threads for `experience` (a registrar seizing domains, "what tech
job lets me do the least work"), technical threads for `evidence` (the
Cloudflare 18 Nov 2025 post-mortem, cloud VM benchmarks, WSL3 benchmarks,
Intel vs. AMD benchmarks, the Section 174 tax letter), and opinion-heavy
threads (Terry Tao and AI in math, "software factories", an
LLM-proof programming language). Up to 28 comments per thread. I dropped bare
links, comments over 1,500 characters, exact duplicates, and comments that
were only questions. Of 322 collected, 29 more were skipped while labelling
(questions with a sentence of context, fragments, typo reports), leaving 293.

**How I labelled them:** All 293 labels were assigned by Claude using my
definitions and decision rules. I did **not** do a cold pass and did not
check the labels row by row, so there are no rows marked `cold`. The `note`
column says `pre-labelled by Claude, not human-checked` on every row, and
posts that were hard calls say which two labels they sat between (13 rows).
This means the labels measure how Claude applied my rules, not how I apply
them, and there is no cold set for unit 6's agreement report.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `opinion` | 189 | 64.5% |
| `experience` | 54 | 18.4% |
| `evidence` | 50 | 17.1% |
| **Total** | 293 | 100% |

**Three hard cases**

**1.**
> *The post:* "A few comparisons just from a gaming PC with a 9800X3D … 7 Zip
> benchmark 9800X3D 130 GIPs compression … C8A 21577 MIPs … A desktop class
> CPU is definitely quicker single threaded and multithreaded, no surprises
> there"
>
> *Could have been:* `experience` or `evidence`
>
> *I chose, because:* `experience`. It is full of hard numbers, but they come
> from the writer's own machine, so the only way to check them is to trust
> the writer. That's the main decision rule exactly: numbers don't make it
> `evidence` if they're from the writer's own runs.

**2.**
> *The post:* "in my country, the Netherlands, part-time work is more and more
> becoming the standard. Already, 75% of women and 30% of men work 32 hours a
> week or less. … I myself started on 40 and will move to 32 hours somewhere
> next year. It's your right by law to reduce your hours"
>
> *Could have been:* `evidence` or `experience`
>
> *I chose, because:* `evidence`. It has both a public statistic and a
> personal story, so I used the tie-break: which one does the advice rest on?
> "It's very common here and won't be a roadblock" rests on the national
> figures and the law, not on the writer's own hours.

**3.**
> *The post:* "This is an anonymous post on Reddit. I repeat. This was on
> Reddit. Consider what credibility there might be."
>
> *Could have been:* `evidence` or `opinion`
>
> *I chose, because:* `evidence`. It is short and sounds like a dismissal,
> but the fact it states (the source is an anonymous Reddit post) is
> checkable and is the entire argument. Tone doesn't decide it; whether a
> stated fact is doing the work does.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42. Trained on an Apple Silicon GPU (`mps`, torch 2.14.1), as
recorded in `results.json`.

**Anything I changed from the defaults, and why:** Nothing. The only edit to
the notebook was `LABELS = ["experience", "evidence", "opinion"]`. One thing
to know: some HN comments run well past 128 tokens (I kept posts up to 1,500
characters), so the model only sees the start of the longer ones.

**Split sizes:** 205 train / 44 val / 44 test (70/15/15, stratified).

| Label | Train | Val | Test |
|---|---|---|---|
| `experience` | 38 | 8 | 8 |
| `evidence` | 35 | 7 | 8 |
| `opinion` | 132 | 29 | 28 |

`experience` and `evidence` have exactly 8 each in test, right at the edge
where per-label F1 gets jumpy: one post is 12.5% of that label's recall.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Overall:** I did this project with Claude (Claude Code) doing most of the
hands-on work, and it should be read that way. Claude picked Hacker News
after Reddit blocked automated access, read the Milestone 1 threads and
wrote the candidate distinctions, drafted the three labels, definitions,
examples and decision rules, collected the posts through the HN API, assigned
every label, and ran the notebook. I chose the five criteria targets myself
from options Claude laid out.

**Moment 1**

- *What I asked for:* A stress test of the draft labels: eight made-up HN
  comments that sit on the boundaries between them, each classified under
  the draft definitions.
- *What came back:* Eight comments (see the stress-test table under Label
  Taxonomy). Four couldn't be classified cleanly under the draft definitions.
- *What changed:* `evidence` now requires the fact to be stated and doing
  work, not just a source named. `experience` now requires a specific event
  with an outcome beyond the writer's feelings, which moved "I tried X and
  hated it" and "everyone I know got burned" into `opinion`.

**Moment 2**

- *What I asked for:* Help with `criteria.md`. I chose the area and the
  target number for each of the five criteria myself (macro F1 0.55,
  per-label F1 0.40, 70% balance cap, +10 points confidence gap, 70% staff
  agreement), and Claude wrote the wording and the "why" paragraphs from
  facts about my data.
- *What came back:* Five criteria in `criteria.md`, each tied to a metric the
  notebook or `agreement.py` reports.
- *What I changed:* Nothing. I kept the wording as drafted.

**Pre-labelling disclosure:** All 293 rows were labelled by Claude from my
definitions and decision rules, and I did not check them row by row or label
a cold set myself. Claude also collected the posts through the HN API.

**Unit 6, Milestone 1:** Claude ran `baseline.py` with my definitions, with
bare label names, and with `--trained results.json`. It committed
`baseline_results.json` and `baseline_bare.json`, and wrote the Baseline vs.
Trained tables and the "What the gap actually means" paragraph from those
results. I chose to see the results before writing a prediction, so there is
none.

**Unit 6, Milestones 2 and 3:** Claude ran notebook sections 1 and 6–7 from a
script (not inside Jupyter, and without re-running section 4, so
`results.json` was not overwritten), committed
`results_three_seeds_before.json`, and wrote the Run Log tables and the
explanations under them. For Milestone 3, I gave Claude the staff labels
file before any labelling, then asked it to label the 30 staff posts anyway. Claude
had already seen the answer key when it labelled them, so the 100% agreement
is not a blind result. Claude also wrote the Agreement Report.

**Unit 6, Milestone 4:** Claude wrote the verdicts and diagnoses from the
three-seed results, the training log and the baseline predictions, including
the argument for the opposite verdict on criterion 4.

**Unit 6, Milestone 5:** Claude chose the change (EPOCHS 3 → 10) from the
Milestone 4 diagnosis, edited the notebook, retrained the three seeds from a
script, committed `results_three_seeds_after.json`, and wrote The
Improvement section.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.636 | 0.636 | +0.000 |
| Macro F1 | 0.390 | 0.259 | -0.130 |
| F1 — `evidence` | 0.400 | 0.000 | -0.400 |
| F1 — `experience` | 0.000 | 0.000 | +0.000 |
| F1 — `opinion` | 0.769 | 0.778 | +0.009 |

Both columns are on the same 44 held-out posts in `test_split.csv`. The
baseline is `facebook/bart-large-mnli` zero-shot, prompted with the
definitions in `label_definitions.txt` (`baseline_results.json`).

**What the definitions were worth.** I also ran the baseline with bare label
names only (`baseline_bare.json`). The baseline reported above is the
definitions run.

| Measure | Bare names | Definitions | Difference |
|---|---|---|---|
| Overall accuracy | 0.432 | 0.636 | +0.205 |
| Macro F1 | 0.319 | 0.390 | +0.071 |
| F1 — `evidence` | 0.000 | 0.400 | +0.400 |
| F1 — `experience` | 0.400 | 0.000 | -0.400 |
| F1 — `opinion` | 0.558 | 0.769 | +0.211 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

I did not write a prediction. I saw the baseline and trained numbers before
writing one, so any prediction written now would be made after the fact.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

Fine-tuning added nothing on this run. The untrained baseline beats it on
macro F1 (0.390 vs. 0.259), and that is the number that matters here.

The tied accuracy (0.636) hides that difference. The trained model reaches
0.636 by predicting `opinion` for all 44 posts, which is exactly the share of
`opinion` in the test set (28 of 44). The baseline also predicts `opinion`
for most posts (37 of 44), but it calls 7 posts `evidence` and 3 of those
are right. That is the whole gap: an `evidence` F1 of 0.400 against 0.000.

Neither model finds `experience`. The trained model never predicts it, and
the baseline with my definitions never predicts it either. Of the 8
`experience` posts, the baseline put 7 under `opinion` and 1 under `evidence`.
The bare-names run did predict `experience`, but for 27 of 44 posts, so its
`experience` F1 of 0.400 comes from over-guessing the label, not from telling
it apart. Since two different models both fail on `experience`, the problem
may be in the label itself or in how it was applied in `labels.csv`, not only
in training. The staff agreement check is where I can test that.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Macro F1 | ≥ 0.55 | 0.259 | 0.259 | 0.259 | MISSED |
| 2. Every label's F1 | ≥ 0.40 | 0.000 | 0.000 | 0.000 | MISSED (`experience` and `evidence` are 0 on every seed) |
| 3. Largest label's share of `labels.csv` | ≤ 70% | 64.5% | 64.5% | 64.5% | MET (dataset-level, so the same on every seed) |
| 4. Most-sure third minus least-sure third | ≥ +10 points | +35.7 | +64.3 | +57.1 | MET |
| 5. Agreement with staff labels | ≥ 21 of 30 | 30 / 30 | — | — | NOT MET: no valid measurement, because the labels were made after the answer key was seen (see Agreement Report) |

All three seeds ran on the same device: Apple Silicon GPU (MPS), torch
2.14.1, `max_length` 128, 3 epochs, learning rate 2e-5, batch size 16. Each
test split had 44 posts (28 `opinion`, 8 `experience`, 8 `evidence`). The
numbers are in `results_three_seeds_before.json`.

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.636 | 0.636 | 0.636 | 0.000 |
| Macro F1 | 0.259 | 0.259 | 0.259 | 0.000 |
| F1 — `experience` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `evidence` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `opinion` | 0.778 | 0.778 | 0.778 | 0.000 |

**The zero spread is not stability.** The notebook calls a 0.000 spread
"stable enough to trust", but here it only means that all three models
predicted `opinion` for every post. The splits are stratified, so every test
set has 28 `opinion` posts out of 44, and a model that always answers
`opinion` scores exactly 28/44 = 0.636 on any of them. The seeds agree
because the model collapsed the same way three times, not because the
measurement is reliable.

Criterion 4 passes, but it measures something narrow here. Since every
prediction is `opinion`, a "right" answer just means the post really is
`opinion`. The gap says the model is more confident on real `opinion` posts
than on the others, so it does pick up some signal, but not enough to change
a single prediction.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42. Seeds 7 and 2024 produced the identical matrix.

| true \ predicted | experience | evidence | opinion |
|---|---|---|---|
| **experience** | 0 | 0 | 8 |
| **evidence** | 0 | 0 | 8 |
| **opinion** | 0 | 0 | 28 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

All 8 real `experience` posts were called `opinion`, and 0 went the other way.
The same is true of all 8 `evidence` posts, so the two cells tie at 8. This
is not one boundary the model failed to learn: it learned no boundary at all
and put every post under the majority label. The direction toward `opinion`
fits the imbalance (65% of `labels.csv` is `opinion`), and the agreement check
in Milestone 3 is where I can test whether my labels are also part of it.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Macro F1 on the test set, all three seeds | ≥ 0.55 | **MISSED** | 0.259 on every seed, less than half the target. No reading of three identical numbers reaches 0.55. |
| 2 | Every label's F1, all three seeds | ≥ 0.40 | **MISSED** | `opinion` is 0.778 on every seed, but `experience` and `evidence` are 0.000 on every seed. |
| 3 | Largest label's share of `labels.csv` | ≤ 70% | **MET** | `opinion` is 189 of 293 = 64.5%. This is a property of the dataset, not of a seed, so one number covers all three. |
| 4 | Most-sure third beats least-sure third, all three seeds | ≥ +10 points | **MET** | +35.7, +64.3 and +57.1 points. The lowest is more than three times the target. See the caveat below. |
| 5 | Agreement with staff labels | ≥ 21 of 30 | **NOT MET** | The script reports 30/30, but the labels were made after the answer key was seen, so the number doesn't measure what the criterion asks. A criterion with no valid measurement can't be called MET. |

**Arguing the opposite verdict on criterion 4.** The strongest case for
MISSED: the criterion was meant to show that borderline `experience` vs.
`evidence` posts sit in the least-confident third. But the model never
predicts either label, so that boundary is never tested. Every prediction is
`opinion`, so "right" just means "the post really is `opinion`". The gap shows
the model is a little more confident on real `opinion` posts, nothing more.
I'm keeping MET because the criterion as written is measurable and passes on
all three seeds. But the criterion has a hole: a model that has collapsed to
one label can still pass it. I'm noting that here rather than revising the
criterion, because the criterion wasn't unmeasurable, it was just met in a
way I didn't anticipate.

**Diagnoses**

**Criteria 1 and 2 (macro F1, and the F1 for `experience` and `evidence`) share one cause: the model never moved off the majority label.** The evidence:

- **It predicts `opinion` for every post, on every seed.** All three
  confusion matrices are identical: 8 → `opinion`, 8 → `opinion`, 28 →
  `opinion`. Changing the split changed nothing, so this isn't bad luck with
  one split.
- **Its confidence is just the label share.** `opinion` is 132 of 205
  training posts (64.4%). The model's mean confidence is 0.61–0.67 on every
  seed, which is about that share. It is outputting roughly "64% `opinion`"
  for everything instead of reading the post.
- **Training stopped before it got anywhere.** Validation accuracy was 0.659
  (the `opinion` share of the validation set) after every one of the 3
  epochs on all three seeds, while validation loss was still falling
  (0.88 → 0.85 → 0.84 on seed 42). With 205 training posts and batch size
  16, three epochs is only 39 weight updates at learning rate 2e-5. The loss
  curve shows the model was still learning when training stopped.

So of the four common causes, the main one is **too few examples for the
small labels**, together with how short training was. About 37 training
posts each for `experience` and `evidence`, against 132 for `opinion`, means
predicting the majority label is the quickest way to lower the loss, and 39
updates was not enough to get past that. The fix to try in Milestone 5 is
weighting the classes, training for more epochs, or both.

**A genuinely hard label pair may also be part of it, for `experience`.** The
untrained baseline, given only my definitions, also never predicted
`experience` (7 of the 8 real `experience` posts went to `opinion`). So two
different models fail on that label in the same direction, which suggests the
`experience`/`opinion` boundary is hard to see from the text alone, not just
from too little data. I can't tell which of those matters more yet.

**What I can't rule out: inconsistent labels.** The agreement check is the
tool that could tell a labelling problem from a model problem, and mine is
not valid (see Agreement Report). In addition, all 293 labels in `labels.csv`
were applied by Claude with no cold set checked by me. So I have no evidence
either way on whether the `experience`/`opinion` and `evidence`/`opinion`
boundaries were applied consistently. The pattern in the matrix (everything
going one way, to the biggest label) fits the imbalance explanation better
than a labelling one, since inconsistent labels usually show up as errors in
both directions. But that is an inference, not a measurement.

**Criterion 5: missed for a process reason, not a model reason.** The cause
is that the staff labels were seen before labelling, so the check was never
run blind. Nothing about the model or the data explains it.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100% (`agreement_results.json`)

**⚠️ This rate is not valid evidence.** The milestone says to label the 30
posts before opening `data/staff_labels.csv`. That did not happen. I gave Claude
the staff labels file first, and then Claude labelled all 30 posts at my
request, after the answer key had already been seen. A labeller who has read
the key cannot label blind, so 100% here shows that the labels match the key.
It says nothing about whether the staff rules can be applied consistently,
and nothing about whether the 293 labels in `labels.csv` are consistent. I am
not counting this as a pass on criterion 5.

**Disagreements**

None. With no disagreements there is nothing to adjudicate or sort into the
three piles.

**What the pattern in my disagreements tells me:**

Nothing reliable, because the check was not run blind. That matters for
Milestone 4. The agreement check was the only tool that could separate a
labelling problem from a model problem, and it can't do that here. So the
question raised in Milestone 2, whether the `experience` → `opinion`
collapse comes partly from inconsistent labels in `labels.csv`, is still
open. Those labels were also applied by Claude, without a cold set (see the
pre-labelling disclosure above), so there is no independent check on them
either.

---

## The Improvement

**What I changed:** One training setting. `EPOCHS` went from 3 to 10 in
notebook section 2. Nothing else changed: same 293 labels, same three seeds
(42, 7, 2024), same splits, learning rate 2e-5, batch size 16,
`max_length` 128, same device (Apple Silicon GPU, MPS). Section 6 ran with
`RUN_LABEL = "after"`, so the before numbers in
`results_three_seeds_before.json` were not overwritten. The new numbers are
in `results_three_seeds_after.json`.

**Which diagnosis pointed at it:** The diagnosis for criteria 1 and 2 found
that training stopped before the model got anywhere. Validation loss was
still falling at epoch 3 on every seed (0.88 → 0.85 → 0.84 on seed 42),
while validation accuracy stayed fixed at the `opinion` share. Three epochs
was only 39 weight updates. The diagnosis also named too few examples for
the small labels. I chose more epochs because it is one setting, it tests the
"stopped too early" part directly, and it keeps the labels unchanged, so any
difference comes from training and not the data.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Macro F1 | ≥ 0.55 | 0.536 | 0.496 | 0.565 | MISSED (only seed 2024 reaches it) |
| 2. Every label's F1 (lowest label shown) | ≥ 0.40 | 0.200 (`evidence`) | 0.182 (`experience`) | 0.364 (`evidence`) | MISSED |
| 3. Largest label's share of `labels.csv` | ≤ 70% | 64.5% | 64.5% | 64.5% | MET (data unchanged) |
| 4. Most-sure third minus least-sure third | ≥ +10 points | +42.9 | +42.9 | +28.6 | MET |
| 5. Agreement with staff labels | ≥ 21 of 30 | 30 / 30 | — | — | NOT MET: still no valid measurement (unchanged by this run) |

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.705 | 0.705 | 0.727 | 0.023 |
| Macro F1 | 0.536 | 0.496 | 0.565 | 0.069 |
| F1 — `experience` | 0.588 | 0.182 | 0.500 | 0.406 |
| F1 — `evidence` | 0.200 | 0.462 | 0.364 | 0.262 |
| F1 — `opinion` | 0.820 | 0.844 | 0.831 | 0.024 |

Confusion matrix after, seed 42:

| true \ predicted | experience | evidence | opinion |
|---|---|---|---|
| **experience** | 5 | 0 | 3 |
| **evidence** | 2 | 1 | 5 |
| **opinion** | 2 | 1 | 25 |

**Did it help, and how do I know:**

Yes, a lot, but not enough to change a verdict. Before vs. after, side by side:

| Measure | Before (3 epochs), all seeds | After (10 epochs): 42 / 7 / 2024 |
|---|---|---|
| Overall accuracy | 0.636 | 0.705 / 0.705 / 0.727 |
| Macro F1 | 0.259 | 0.536 / 0.496 / 0.565 |
| F1 — `experience` | 0.000 | 0.588 / 0.182 / 0.500 |
| F1 — `evidence` | 0.000 | 0.200 / 0.462 / 0.364 |
| F1 — `opinion` | 0.778 | 0.820 / 0.844 / 0.831 |

- **The collapse is gone.** Before, every seed predicted `opinion` for all 44
  posts. After, every seed predicts all three labels, and macro F1 roughly
  doubled on every seed (0.259 → 0.496–0.565).
- **It now beats the baseline.** The untrained baseline scored 0.390 macro F1
  on the seed 42 test split. The retrained model scores 0.536 on that same
  split, so fine-tuning now adds something it didn't before.
- **It still misses criteria 1 and 2.** Macro F1 reaches 0.55 on one seed
  out of three. The weakest label is below 0.40 on every seed, and it isn't
  the same label each time (`evidence` on seeds 42 and 2024, `experience` on
  seed 7).
- **The small labels are now the unstable part.** `experience` F1 ranges
  from 0.182 to 0.588 across seeds, a spread of 0.406. With 8 test posts per
  small label, one post moves F1 a lot. That matches the "too few examples"
  half of the diagnosis, which more epochs can't fix.
- **The remaining errors still lean toward `opinion`.** On seed 42, 5 of 8
  `evidence` posts and 3 of 8 `experience` posts went to `opinion`, while only
  3 `opinion` posts went the other way. The lean is smaller than before but
  in the same direction.

In one sentence: training for 10 epochs instead of 3 fixed the collapse to
`opinion` and doubled macro F1, but it did not lift either missed criterion
over its target on all three seeds.

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
