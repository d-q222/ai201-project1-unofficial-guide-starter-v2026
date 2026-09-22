# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

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

This is a retrieval-augmented generation (RAG) system that splits documents from a corpus into chunks and scores them on relevance before feeding them to an LLM as sources to respond to a question. I picked the "campus life" corpus. My system answers questions about student life, including the workloads of various classes, wait times in dining halls, and grading and administrative policies. This system is tuned to respond to questions only when the answer to the question is clearly stated in the corpus. 
<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:**
Each sentence is about 100 characters long, so this is a good starting point for the "campus life" corpus. They are also short posts, so each sentence contains answers to some question. 

However, this ended up with an incoherent jumble of chunks, so I decided the fallback would be 800, as I chunked instead by paragraphs and included the header at the top of each. If the more advanced chunking were to fail, then I would just have a whole document as one chunk.

**Overlap:**
Overlap was 50, since I hoped to pick up some of the characters that may be straggling since any set number is a little too exact.

After I changed the fallback to 800, the overlap doesn't matter anymore sice 800 covers basically all of the text for every doc.

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
"What do students say about the wait times of Pellew Dining Hall"
**Answer:**

```
(best distance 0.197, cutoff 0.45)

Students state that the wait times at Pellew Dining Hall are 12 to 18 minutes at peak, which occurs early from 11:45 to 12:30. *(dining_pellew_dining_hall.txt, dining_pellew_dining_hall_followup.txt)*

Sources retrieved: dining_halden_hall.txt, dining_halden_hall_followup.txt, dining_pellew_dining_hall.txt, dining_pellew_dining_hall_followup.txt

1 model calls this session, 683 tokens (614 in, 69 out)
```

**My relevance cutoff:**

My relevance cutoff was 0.45. Of all my test questions, the best distances were 0.197, 0.219, 0.359, 0.328, and 0.380. For the given bogus questions, the best distances were 0.825, 0.886, 0.844, 0.896, and 0.934. My relevance cutoff sits in the gap between them

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
|"What is the deadline for grade appeals?"| Yes | 0.219 |
|"What is the format for the ECON 101 final?"|Yes|0.359|
|"What do students say about PHYS 130?"|Yes|0.328|
|"What is the workload for ENGL 205?"|Yes|0.380|
|"What do students say about the wait times of Pellew Dining Hall"|Yes|0.197|
|"How do I write a for loop in Rust?"|No|0.896|
|"What is the recommended dosage of ibuprofen for a headache?"|No|0.844|
|"Who won the 1994 World Cup?"|No|0.886|
|"How do I change the oil in a diesel engine?"|No|0.934|
|"What is the capital of Mongolia?" |No|0.825|

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**
I asked Codex Luna to review my chunking-by-paragraph code, as the chunks were returning only the titles. It gave me two suggestions: instead of searching for line breaks, search for paragraphing, and make sure my start index is advanced sufficiently. I implemented both of these suggestions myself and caught some more bug fixes in the process. 
**2.**
I asked Claude to see if my chunking-by-paragraph method made chunks that stood alone. It told me that my first paragraph happened to stand alone since I had decided to add the header with the first paragraph, but the others needed some inference. It suggested that I include headers in every single chunk, and I implemented that code by myself. 

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

## Stretch Features
Metadata filtering: Added a source query that can filter responses by source.

|Question|Before|After|
|---|---|---|
|"What is the workload for ENGL 205?"|(best distance 0.380, cutoff 0.6) The workload for ENGL 205 is 4 to 5 hours a week, mostly spent on writing and rewriting (source: `course_engl_205_workload.txt` and `course_engl_205.txt`). It is front-loaded, meaning the first month is heavier than the rest as you learn the format (`course_engl_205_workload.txt`). Sources retrieved: course_cs_210_workload.txt, course_engl_205.txt, course_engl_205_workload.txt, course_phys_130_workload.txt \n 1 model calls this session, 575 tokens (487 in, 88 out)|(best distance 0.468, cutoff 0.5). The workload for ENGL 205 is expected to be 4 to 5 hours a week, which is mostly spent on writing and rewriting (course_engl_205.txt). \n Sources retrieved: course_engl_205.txt \n 1 model calls this session, 313 tokens (272 in, 41 out)|

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

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

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

## The Improvement

**What I changed:**

**Why I picked it:**

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
