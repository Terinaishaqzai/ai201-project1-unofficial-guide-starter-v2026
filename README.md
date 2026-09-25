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

This is a RAG system built on the `campus_life` corpus — 88 short posts covering housing, courses, dining, and campus admin topics that students actually ask each other about. It answers specific questions like "is the housing lottery random?" or "how many unit tests are there in BIOL 160?" by retrieving the most relevant chunks from those posts and generating an answer grounded only in what it finds, always naming its source. If a question falls outside what the corpus covers — like general trivia or unrelated how-to questions — a relevance gate catches it and the system says so instead of guessing.

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


**1.** I asked Claude to write a paragraph-based chunker to replace the starter's fixed-size one, since my documents bundle several sub-topics per file. The first version merged short paragraphs backward into the previous one, but that missed short leading title lines with no previous chunk to merge into — my index run showed a 10-character chunk as a result. I pointed this out, and Claude rewrote the merge logic to accumulate forward instead. Re-running the index confirmed the shortest chunk became exactly 100 characters.

**2.** I asked Claude to help me draft my 5 test questions and 2 extra acceptance criteria based on documents I'd read from my corpus. When reviewing sample chunks, Claude flagged that one of my 5 sample chunks ("Also worth saying: one register...") assumed context from an earlier paragraph in the same file. Rather than swapping it for a cleaner example, I decided to keep it and note the limitation directly in my README, since an honest imperfect example felt more useful than a cherry-picked perfect one.


# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks are complete thoughts, 100–600 characters | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Known source is the #1 ranked chunk | 3 of 5 | 5/5 | 5/5 | 5/5 | MET |

The three answers per question are saved in `results/run_2026-09-24_1537_before.md`. Retrieval and chunking gave the same results across runs. The gate was tested once because it is deterministic, so its result appears in all three columns.

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->
All five criteria met their targets. My targets were fairly safe: criterion 4 only required 4 of 5 sampled chunks to stand alone, and that is exactly what I got. I would tighten it to 5 of 5 in a future test.

The remaining weakness is in chunking. The Verrill Street Grill chunk starts “Also worth saying,” referring to an earlier paragraph. My chunker split at the paragraph break, leaving that chunk without its full context.


## The Improvement

**What I changed:** I updated `chunker.py::split_documents` to keep follow-up paragraphs with the preceding chunk when the combined text is at most 600 characters. The Verrill Street Grill follow-up now appears with the earlier paragraph instead of alone.

**Why I picked it:** My diagnosis found that paragraph splitting removed the context needed to understand “Also worth saying.”

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->