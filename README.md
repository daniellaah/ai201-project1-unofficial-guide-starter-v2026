# The Unofficial Guide

**Author:** Bo Gao

**Corpus:** `campus_life`

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This project makes the `campus_life` corpus of 88 short student posts
searchable, covering topics such as courses, administrative deadlines, health
services, campus jobs, dining, and housing. It divides each document into
paragraph-sized chunks, repeats the document title for context, and retrieves
the three chunks most closely related to a question. The system generates a
brief answer using only the retrieved documents and names the source file it
used. A relevance gate with a `0.6` cutoff refuses questions that the corpus
does not contain enough information to answer.

## Chunking Strategy

**Chunk size:** One body paragraph, with the document title prepended. In the
current corpus this produces chunks from 63 to 397 characters, with an average
of 167 characters.

**Overlap:** Zero body-text characters. The document title is repeated in each
chunk so that every paragraph keeps its subject during embedding and retrieval.

The baseline 800-character window produced one chunk per document because all
88 `campus_life` documents were shorter than 800 characters. Most posts are
short, but the longer housing posts contain separate paragraphs about rooms,
advantages, disadvantages, laundry, and noise. Splitting on blank lines keeps
complete sentences together while making each chunk focus on a smaller topic.

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_cs_340_exams.txt#1` — produced by: `chunker.py::split_documents`

```
CS 340 Databases — assessment

Start the term project in week three, not week eight; everyone learns this the hard way.
```

**Chunk 3** — source: `course_phys_130_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for PHYS 130 Mechanics

People keep asking so: 7 hours a week, plus 3 on lab weeks. That's real time, not optimistic time.
```

**Chunk 4** — source: `dining_verrill_street_grill_followup.txt#1` — produced by: `chunker.py::split_documents`

```
Re: Verrill Street Grill

Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_morrow_house.txt#1` — produced by: `chunker.py::split_documents`

```
Morrow House — what it's actually like

The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**

What are the health center's walk-in hours?

**Answer:**

```
The health centre's walk-in hours are from 8am to 11am (health_center.txt).

Sources retrieved: dining_the_atrium.txt, health_center.txt
```

**My relevance cutoff:** `0.6`

The five in-corpus questions had best distances from `0.1865` to `0.3599`,
while the five out-of-scope questions ranged from `0.7873` to `0.9228`. The
gap between the two groups was `0.4274`, so I kept the `0.6` cutoff inside that
gap. At this cutoff all five in-corpus questions passed the relevance gate and
all five out-of-scope questions were refused.

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question                                                                    | In corpus? | Best distance |
| --------------------------------------------------------------------------- | ---------- | ------------- |
| How are lab problems used in the CS 210 exams?                              | Yes        | 0.1865        |
| What is the deadline for dropping a course?                                 | Yes        | 0.2597        |
| What are the health center's walk-in hours?                                 | Yes        | 0.2120        |
| Can students usually study during a library desk shift?                     | Yes        | 0.3599        |
| What is the best time to do laundry to avoid waiting for a washer or dryer? | Yes        | 0.3115        |
| What is the capital of Mongolia?                                            | No         | 0.7873        |
| How do I change the oil in a diesel engine?                                 | No         | 0.9228        |
| Who won the 1994 World Cup?                                                 | No         | 0.8474        |
| What is the recommended dosage of ibuprofen for a headache?                 | No         | 0.8487        |
| How do I write a for loop in Rust?                                          | No         | 0.8598        |

## How I Used AI

**1.** I asked AI to pressure-test my acceptance criteria rather than write
them from scratch. My original chunking criterion required every chunk to be
shorter than 300 characters, but I could not justify that number. The AI
pointed out that tiny sentence fragments would still pass that test, so I
changed the criterion to require that no chunk begin or end in the middle of a
sentence, then used AI feedback to make the English precise and testable.

**2.** I asked AI whether splitting housing posts into sections such as “The
good,” “The bad,” “Laundry,” and “Noise” would make the chunks lose their
subject. After inspecting the pipeline, it explained that source metadata is
shown to the answer model only after retrieval, while embeddings are created
from the chunk text itself. I therefore changed the chunker to prepend the
document title to every paragraph-sized chunk, then verified all 183 chunks
used original paragraph boundaries and inspected sample chunks before keeping
the strategy.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

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

| Criterion                              | Target | Run 1 | Run 2 | Run 3 | Verdict |
| -------------------------------------- | ------ | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer | 4 of 5 |       |       |       |         |
| 2. Every answer names a source         | 5 of 5 |       |       |       |         |
| 3. Gate stops out-of-corpus questions  | 4 of 5 |       |       |       |         |
| 4.                                     |        |       |       |       |         |
| 5.                                     |        |       |       |       |         |

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

| #   | Criterion | Verdict | How I decided |
| --- | --------- | ------- | ------------- |
| 1   |           |         |               |
| 2   |           |         |               |
| 3   |           |         |               |
| 4   |           |         |               |
| 5   |           |         |               |

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

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion                              | Target | Run 1 | Run 2 | Run 3 | Verdict |
| -------------------------------------- | ------ | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer | 4 of 5 |       |       |       |         |
| 2. Every answer names a source         | 5 of 5 |       |       |       |         |
| 3. Gate stops out-of-corpus questions  | 4 of 5 |       |       |       |         |
| 4.                                     |        |       |       |       |         |
| 5.                                     |        |       |       |       |         |

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
