# The Unofficial Guide

Terina Ishaqzai — campus_life corpus

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Milestone 5. -->

## Chunking Strategy

**Chunk size:** Not a fixed size — chunks follow paragraph boundaries.
**Overlap:** None. Paragraph splitting doesn't need overlap the way a fixed-size window does, since each chunk already stops at a natural break instead of cutting mid-sentence.

My corpus (`campus_life`) is made up of short student posts, but many of them
bundle several distinct sub-topics into one file as separate paragraphs — a
housing post typically has 5 paragraphs (intro, pros, cons, laundry, noise),
and a course post has 4 (intro, format, workload, exam pattern). The starter's
fixed 800-character chunker never split anything on this corpus (88 documents
→ 88 chunks), because no document even reaches 800 characters — but that
doesn't mean one post is really one thought. It's several.

I replaced it with a chunker that splits on paragraph breaks (`chunker.py::split_documents`), so each sub-topic becomes its own chunk. Paragraphs under 100 characters (including short title lines) get merged forward into the next paragraph so no chunk is left as a meaningless fragment. This turned 88 documents into 143 chunks, averaging 194 characters (range: 100–409), down from the fallback's 317-character average.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.


**Chunk 2** — source: `course_cs_340.txt#0` — produced by: `chunker.py::split_documents`

CS 340 Databases

I'm a junior and I've done this twice now. Format is lecture twice a week plus a project that runs the whole term. Assessment: one midterm and a final, both open-book. Lightly curved, usually two or three points.


**Chunk 3** — source: `course_phys_130_exams.txt#0` — produced by: `chunker.py::split_documents`

PHYS 130 Mechanics — assessment

Three midterms, no final, plus a lab practical. Not curved, but the lowest midterm is dropped.

The lab practical is worth 20% and almost nobody prepares for it.


**Chunk 4** — source: `dining_verrill_street_grill_followup.txt#1` — produced by: `chunker.py::split_documents`

Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.


*Note: this chunk opens with "also worth saying," which implies a point made earlier in the same file. It reads fine on its own, but leans slightly on context it no longer has access to — a real limitation of splitting purely on paragraph breaks rather than by topic.*

**Chunk 5** — source: `housing_morrow_house.txt#1` — produced by: `chunker.py::split_documents`

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.


## Sample Answer

<!-- Milestone 4. -->

**Question:** is the housing lottery random?

**Answer:**
The housing lottery is not entirely random; rising sophomores get a number drawn at random, but juniors and seniors are ordered by accumulated credit hours first, using random selection only as a tie-breaker (admin_housing_lottery.txt).
**My relevance cutoff:** 0.6 (the starter's default). I ran all 5 of my test questions and all 5 OUT_OF_SCOPE questions and found a large, clean gap: my in-scope questions topped out at 0.363, and my out-of-scope questions bottomed out at 0.803. 0.6 sits comfortably in the middle of that gap, so I kept the default rather than moving it.

<!-- Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery actually random? | Yes | 0.254 |
| What's the noise situation like in Morrow House? | Yes | 0.245 |
| How many unit tests are there in BIOL 160? | Yes | 0.260 |
| If I have dining dollars left over in May, do I keep them for next fall? | Yes | 0.317 |
| Who do I talk to first if I want to appeal a grade? | Yes | 0.363 |
| What is the capital of Mongolia? | No | 0.825 |
| How do I change the oil in a diesel engine? | No | 0.923 |
| Who won the 1994 World Cup? | No | 0.874 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.803 |
| How do I write a for loop in Rust? | No | 0.877 |


## How I Used AI

<!-- Milestone 5. -->

**1.**

**2.**

---

# Unit 2

## Run Log — Before

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

## The Improvement

**What I changed:**

**Why I picked it:**

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

## What's Still Broken

## What I'd Do Differently
