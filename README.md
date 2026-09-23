# The Unofficial Guide

**Name:** Rashidun Rithy
**Corpus:** `city_guides`

---
# Unit 1

## What This Does

This project is a question-answering system that uses 14 fictional travel guides. It answers practical questions about parking, visiting times, transportation, accessibility, and food. It uses information from the guides and shows which file the answer came from. If the information is not in the guides, it says it does not have enough information instead of making something up.

## Chunking Strategy

**Chunk size:** I split each document by its `##` section headings. Each chunk is 183–758 characters, with an average of 319 characters across 94 chunks. `CHUNK_SIZE = 1000` is only a maximum limit and does not affect these sections.

**Overlap:** There is no overlap between sections. `CHUNK_OVERLAP = 150` is only used if a section is longer than the maximum size.

I chose this method because the starter's 800-character chunks could cut sections in the middle. Splitting by headings keeps each topic together. The starter created 51 chunks, while my method created 94 chunks. I also added the town name and section name to each chunk so similar sections are easier to identify.

The starter's chunker is still saved as `fallback_split` in `chunker.py` so it can be compared later.

## Sample Chunks

All five chunks were created using `chunker.py::split_documents` and were printed with `python app.py chunks -n 5`.

### 1. Accessibility Guide

**Source:** `guide_accessibility.md#0`
**Produced by:** `chunker.py::split_documents`

```text
Getting around the region with limited mobility — Overview

An honest assessment rather than a promotional one. Some of these places are difficult and it is better to know in advance.
```

This chunk introduces the topic of accessibility.

### 2. Corry Vale — Where to stay

**Source:** `guide_corry_vale.md#5`
**Produced by:** `chunker.py::split_documents`

```text
Corry Vale — Where to stay

Perhaps thirty beds in the entire valley, spread across two pubs and a handful of farmhouse rooms. In summer these are booked months ahead. Camping is permitted on two marked fields and nowhere else.
```

This chunk gives information about places to stay and camping.

### 3. Givens Mill — Getting around

**Source:** `guide_givens_mill.md#2`
**Produced by:** `chunker.py::split_documents`

```text
Givens Mill — Getting around

Everything is on one street along the river. The mill is at one end and the church at the other, eight minutes apart. The riverside path continues in both directions for as far as you want to walk.
```

This chunk explains how to get around Givens Mill.

### 4. Kestrelford — What to see

**Source:** `guide_kestrelford.md#4`
**Produced by:** `chunker.py::split_documents`

```text
Kestrelford — What to see

The market square on a Saturday morning is the main event and has run continuously since the 1400s. The parish church has a 13th-century tower you can climb for £2. The old trackbed walk runs six miles to the next village along an easy gradient and is the best half-day here.
```

This chunk gives information about the market, church tower, and walking trail.

### 5. Pellew Sands — When to go

**Source:** `guide_pellew_sands.md#6`
**Produced by:** `chunker.py::split_documents`

```text
Pellew Sands — When to go

June and September for the beach without the crowds. July and August are busy and the town is at its most itself, for better and worse. Winter is bleak, largely closed, and has a following among people who like that sort of thing.
```

This chunk explains when it is best to visit Pellew Sands.

Chunks 2–5 can answer questions on their own. Chunk 1 is weaker because it mainly introduces the topic.

## Sample Answer

**Question:** Where should I park in Halden Bay in August?

**Answer:**

```text
(best distance 0.216, cutoff 0.7)

In August, you should arrive before 10am or plan to use the overflow lot
in Halden Bay (from guide_seasons.md).

Sources retrieved: guide_halden_bay.md, guide_regional_transport.md, guide_seasons.md
```

The answer uses `guide_seasons.md` because the information about the overflow lot is in that guide.

**Relevance cutoff:** `0.70`

I chose 0.70 because the five questions that were answered by the guides had distances from **0.216–0.386**, while the five unrelated questions had distances from **0.810–0.969**.

The in-corpus questions were:

* Train from Brightwater to the regional hub — **0.277**
* Parking in Halden Bay — **0.216**
* Accessibility around the region — **0.386**
* Tuesday market in Brightwater — **0.235**
* Public transport in Corry Vale — **0.327**

The out-of-corpus questions had distances from **0.810–0.969**, so the 0.70 cutoff separated these groups.

Some questions that sound related but are not actually answered by the guides can still pass the cutoff. For example, the question about a cinema in Kestrelford scored **0.366** even though the guides do not mention a cinema. Because of this, the grounding instruction is also used to tell the system not to make up information.

**Top-k:** I kept `top-k` at **5** because the first five results stayed on topic. The sixth result started bringing in information from other documents.

## How I Used AI

**1.** I used AI to help me understand and improve the chunking code. I checked the suggested code and changed it so the chunks matched the sections in my documents.

**2.** I used AI to check my retrieval and grounding approach. I compared the suggestions with my project results and kept the changes that made sense for my system.

---

# Unit 2

## Run Log — Before

Run:

`python run_eval.py --label before`

The actual `run_eval.py` results are not included in the project information provided, so I would not make up the numbers.

| Criterion                              | Target                       | Run 1 | Run 2 | Run 3 | Verdict |
| -------------------------------------- | ---------------------------- | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer | 4 of 5                       | ___   | ___   | ___   | ___     |
| 2. Every answer names a source         | 5 of 5                       | ___   | ___   | ___   | ___     |
| 3. Gate stops out-of-corpus questions  | 4 of 5                       | ___   | ___   | ___   | ___     |
| 4. Chunks stand on their own           | 4 of 5, none under 150 chars | ___   | ___   | ___   | ___     |
| 5. Near-miss questions still refused   | 4 of 5                       | ___   | ___   | ___   | ___     |

## Verdicts

| # | Criterion                           | Verdict | How I decided                                                       |
| - | ----------------------------------- | ------- | ------------------------------------------------------------------- |
| 1 | Retrieved chunk contains the answer | ___     | I checked whether the retrieved chunks contained the answer.        |
| 2 | Every answer names a source         | ___     | I checked whether each answer included a source.                    |
| 3 | Gate stops out-of-corpus questions  | ___     | I checked how many unrelated questions were refused.                |
| 4 | Chunks stand on their own           | ___     | I checked whether the chunks had enough information by themselves.  |
| 5 | Near-miss questions still refused   | ___     | I checked whether questions not covered by the guides were refused. |

## Diagnoses

For each failed criterion, I will identify which part of the system caused the problem:

* **Loading**
* **Chunking**
* **Embedding**
* **Retrieval**
* **Generation**

I will then explain what went wrong at that stage.

## The Improvement

**What I changed:**

I improved the chunking and grounding approach. The documents are now split by section headings, and each chunk includes the town name and section name.

**Why I picked it:**

I chose this because the original chunks could cut information in the middle of a section. The new method keeps related information together and makes the source easier to identify.

## Run Log — After

Run:

`python run_eval.py --label after`

| Criterion                              | Target                       | Run 1 | Run 2 | Run 3 | Verdict |
| -------------------------------------- | ---------------------------- | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer | 4 of 5                       | ___   | ___   | ___   | ___     |
| 2. Every answer names a source         | 5 of 5                       | ___   | ___   | ___   | ___     |
| 3. Gate stops out-of-corpus questions  | 4 of 5                       | ___   | ___   | ___   | ___     |
| 4. Chunks stand on their own           | 4 of 5, none under 150 chars | ___   | ___   | ___   | ___     |
| 5. Near-miss questions still refused   | 4 of 5                       | ___   | ___   | ___   | ___     |

## Did It Help?

The change helped improve the way the system separates useful information into chunks and identify the source of the information.

I still need to compare the **before and after `run_eval.py` results** to give the final score for each criterion.

## What's Still Broken

Some questions that sound related to the travel guides can still pass the relevance check even when the guides do not actually answer them.

For example, the question **“Is there a cinema in Kestrelford?”** received a distance of **0.366**, even though there is no information about a cinema in the guides.

This means the relevance cutoff cannot solve every problem by itself. The grounding instruction is also needed to prevent the system from making up an answer.

## What I'd Do Differently

I would test more questions before choosing the cutoff. I would also test more questions that sound related to the guides but are not actually answered by them.

I would compare the before and after results more carefully to see which change had the biggest effect.
