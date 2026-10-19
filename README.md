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
