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

**Where the posts came from:**

**How I labelled them:** <!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->

**Counts per label:**

| Label | Count | Share |
|---|---|---|
|  |  |  |
|  |  |  |
|  |  |  |
| **Total** |  | 100% |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

**2.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

**3.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:**

**Settings:** <!-- epochs, learning rate, batch size, seed -->

**Anything I changed from the defaults, and why:**

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* Eight made-up HN comments that sit on the boundaries
  between my three draft labels.
- *What came back:* Eight comments (see the stress-test table under Label
  Taxonomy). Four of them I couldn't classify cleanly under the draft
  definitions.
- *What I changed:* `evidence` now requires the fact to be stated and doing
  work, not just a source named. `experience` now requires a specific event
  with an outcome beyond the writer's feelings, which moved "I tried X and
  hated it" and "everyone I know got burned" into `opinion`.

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Pre-labelling disclosure:**

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
