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

## Run Log — Before

Chunker: `chunker.py::split_documents` at commit `46107ef`, before the improvement. Index: 143 chunks, shortest 100, longest 409.
Results file: `results/run_2026-09-24_1537_before.md`, produced by `run_eval.py::main`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks are complete thoughts, 100–600 characters | 4 of 5 | 3/5 | 3/5 | 3/5 | MISSED |
| 5. Known source is the #1 ranked chunk | 3 of 5 | 5/5 | 5/5 | 5/5 | MET |

Criteria 3, 4 and 5 are deterministic (gate comparison, chunking, retrieval ranking), so one measurement fills all three columns.

### Real output (before)

**Criteria 1 and 2:** run 1 answers from `results/run_2026-09-24_1537_before.md`, produced by `generate.py::answer_from_chunks`.

```
Q: Is the housing lottery actually random?
The housing lottery is not entirely random in the way most people assume. While rising sophomores get a number drawn at random, juniors and seniors are ordered by accumulated credit hours first, with random selection used only as a tie-breaker (*admin_housing_lottery.txt*).

Q: What's the noise situation like in Morrow House?
In Morrow House, it is loud until about 1am on weekends and there are no enforced quiet hours.
Source: housing_morrow_house_noise.txt

Q: How many unit tests are there in BIOL 160?
There are four unit tests in BIOL 160.
Source: course_biol_160.txt (and course_biol_160_exams.txt)

Q: If I have dining dollars left over in May, do I get to keep them for next fall?
No, whatever dining dollars are left in May disappears and does not roll over to the following autumn (admin_dining_dollars.txt).

Q: Who do I talk to first if I want to appeal a grade?
If you want to appeal a grade, you must start with the instructor.
Source: admin_grade_appeals.txt
```

**Criterion 3:** produced by `run_eval.py::check_out_of_scope`, cutoff 0.6.

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.923 | refused |
| Who won the 1994 World Cup? | 0.874 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.803 | refused |
| How do I write a for loop in Rust? | 0.877 | refused |

**Criterion 4:** the 5-chunk sample from `python app.py chunks -n 5` is the same sample shown in Unit 1 → Sample Chunks above, produced by `chunker.py::split_documents`. Complete: chunks 1, 2 and 3. Incomplete: chunk 4 (Verrill follow-up, never names the restaurant) and chunk 5 (Morrow "The good / The bad", never names the dorm).

**Criterion 5:** rank-1 source per question, from `store.py::search`.

```
admin_housing_lottery.txt | Is the housing lottery actually random?
housing_morrow_house_noise.txt | What's the noise situation like in Morrow House?
course_biol_160_exams.txt | How many unit tests are there in BIOL 160?
admin_dining_dollars.txt | If I have dining dollars left over in May, do I get to keep them for next fall?
admin_grade_appeals.txt | Who do I talk to first if I want to appeal a grade?
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All 15 answers contained the expected fact from `questions.py` (credit hours, 1am, four, disappears, instructor), so the answer was in a retrieved chunk every run. |
| 2 | Every answer names a source | MET | All 15 answers named a source file. |
| 3 | Gate stops out-of-corpus questions | MET | Refused 5 of 5. The closest out-of-scope question was 0.803 against a 0.6 cutoff, a wide margin. |
| 4 | Sampled chunks are complete thoughts | MISSED | I judged each chunk by one test: could someone answer a question using only this chunk? 3 of 5 passed. Two never name their subject. Size bounds (100–600) passed. |
| 5 | Known source ranked #1 | MET | The rank-1 chunk came from the correct document for all 5 questions. |

## Diagnoses

**Criterion 4 — Stage: chunking.**
`split_documents` splits on blank lines. In this corpus, the subject of a post (the course, building or dorm) appears only in the first paragraph. Every later paragraph becomes a chunk that has lost its referent. The Verrill follow-up says "Also worth saying: one register…" without naming the restaurant. The Morrow chunk says "The good: cheapest housing tier…" without naming the dorm.

**Pattern:** every incomplete chunk is a non-first paragraph. It's one problem, not two.

**On the criteria I met:** criteria 1, 2, 3 and 5 all scored 5/5, against targets of 4, 5, 4 and 3. Those targets were safe. All five test questions are single-fact lookups where one self-contained chunk holds the whole answer, which is the easiest case for this pipeline. None of them ask about a fact stored in a non-first paragraph, which is exactly where criterion 4 found the weakness. I would tighten criterion 1 by adding questions like "How many registers does Verrill Street Grill have?" and "What does laundry cost in Morrow House?"

## The Improvement

**What I changed:** I updated `chunker.py::split_documents` so that follow-up paragraphs beginning "Also worth saying:", "Another thing:" or "One more thing:" merge into the preceding chunk when the combined text is at most 600 characters (commit `ff6124b`). The Verrill Street Grill follow-up now appears with the earlier paragraph instead of alone.

**Why I picked it:** it targets the criterion 4 failure directly. The before sample included `dining_verrill_street_grill_followup.txt#1`, a chunk that never names the restaurant it describes.

*Separately, commit `ec8c5a6` made `generate.py` retry on 503 "model overloaded" errors. That changes reliability, not answers, so it is not a second system change.*

### Run Log — After

Chunker: `chunker.py::split_documents` at commit `ff6124b`. Index: 138 chunks, shortest 101, longest 409.
Results file: `results/run_2026-09-25_1429_after.md`, produced by `run_eval.py::main`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks are complete thoughts, 100–600 characters | 4 of 5 | 2/5 | 2/5 | 2/5 | MISSED |
| 5. Known source is the #1 ranked chunk | 3 of 5 | 5/5 | 5/5 | 5/5 | MET |

### Real output (after)

**Criteria 1 and 2:** run 1 answers from `results/run_2026-09-25_1429_after.md`, produced by `generate.py::answer_from_chunks`.

```
Q: Is the housing lottery actually random?
The housing lottery is not entirely random in the way most people assume. While rising sophomores get a number drawn at random, juniors and seniors are ordered by accumulated credit hours first, with random selection used only as a tie-breaker.
Source: `admin_housing_lottery.txt`

Q: What's the noise situation like in Morrow House?
In Morrow House, it is loud until about 1am on weekends and there are no enforced quiet hours.
Source: housing_morrow_house_noise.txt

Q: How many unit tests are there in BIOL 160?
There are four unit tests in BIOL 160 (source: course_biol_160_exams.txt and course_biol_160.txt).

Q: If I have dining dollars left over in May, do I get to keep them for next fall?
No, whatever is left in May disappears and does not roll over to the following autumn (admin_dining_dollars.txt).

Q: Who do I talk to first if I want to appeal a grade?
You must talk to the instructor first to start a grade appeal. This comes from the document `admin_grade_appeals.txt`.
```

**Criterion 3:** same as before, except "Who won the 1994 World Cup?" moved from 0.874 to 0.886. It was refused. The gate refused 5 of 5.

**Criterion 4:** the sample from `python app.py chunks -n 5`, produced by `chunker.py::split_documents`.

```
Chunk 1 | admin_add_drop_deadline.txt#0
On the add/drop deadline
You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

Chunk 2 | course_cs_210_workload.txt#1
It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.

Chunk 3 | course_phys_130.txt#0
PHYS 130 Mechanics
Just finished a year in this building. Format is lecture with a compulsory lab that meets fortnightly. Assessment: three midterms, no final, plus a lab practical. Not curved, but the lowest midterm is dropped.

Chunk 4 | health_center.txt#1
Counselling is separate, in the same building, and has its own intake process with a shorter wait than people expect — usually three or four days for a first session.

Chunk 5 | housing_morrow_house.txt#2
Laundry costs $1.50 wash, $1.25 dry, coin or card. On noise: loud until about 1am on weekends, no enforced quiet hours.
```

Complete: chunks 1 and 3. Incomplete: chunk 2 (which course?), chunk 4 (separate from what, in which building?), chunk 5 (which dorm?).

**Criterion 5:** identical to the before output. Same rank-1 source for all 5 questions.

### Did it help?

Partly, and my test could only partly see it.

- **What changed:** the chunk count dropped from 143 to 138. Five orphaned follow-up paragraphs now sit with their context. The Verrill Street Grill "one register" paragraph is now part of `#0`, which names the restaurant:

```
  dining_verrill_street_grill_followup.txt#0:
  Re: Verrill Street Grill

  Adding to what people have said about Verrill Street Grill. The wait figure of up to 30 minutes on Friday evenings matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

  Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.
```

- **What didn't change:** criteria 1, 2, 3 and 5 were identical. Every best distance and every retrieved source list for my five test questions matched to four decimal places, because none of them touch a document with a follow-up paragraph.
- **Why criterion 4 went from 3/5 to 2/5:** this is not evidence the change hurt. `chunks -n 5` samples by position, and merging 5 chunks shifted which chunks were picked, so the before and after samples contain different chunks. My criterion 4 can't compare two chunkers fairly.

## What's Still Broken

**Criterion 4 is still missed.** My fix only catches three trigger phrases. The real problem is every non-first paragraph ("The good: …", "Laundry costs…", "Counselling is separate…", "It's front-loaded…").

- **Next fix:** prefix every chunk with its document's title line, so every chunk names its subject.
- **Why I stopped:** the unit allows one change, and I wanted the follow-up merge measured on its own first.

**Criterion 4's measurement is also weak.** The sample changes whenever the chunk count changes, so it can't compare two chunkers. I'd judge a fixed list of chunk sources instead, or count across all chunks.

## What I'd Do Differently

- **Write the "why this target" reasons for criteria 1–3.** I left them blank in unit 1. Without reasons, I set targets that five single-fact questions were guaranteed to clear.
- **Write harder test questions.** At least two should ask about facts stored in non-first paragraphs, so the test can see chunking problems.
- **Define criterion 4 on a fixed set of chunks,** so it gives comparable numbers across chunker versions.

## How I Used AI — Unit 2

**3.** When `run_eval.py` kept crashing with 503 errors, I used Claude to trace it. `generate()` only retried on 429 rate limits, so a 503 "model overloaded" error raised immediately. I added 503 to the retry condition with a longer backoff (commit `ec8c5a6`). I also used Claude to compare my before and after results files line by line. It pointed out that every distance was identical, which meant my test questions couldn't see my chunking change. It also flagged that my criterion 4 count was inconsistent between my table and my diagnosis. Claude also noticed that my "Why this target" sections for criteria 1–3 were still blank, which is why I list that under What I'd Do Differently.
