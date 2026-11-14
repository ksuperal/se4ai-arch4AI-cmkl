# Week 6 worksheet: serving patterns

**Name:** Kanokjan Singhsuwan
 
**Student ID:** 01223046

This is **individual work**: each student completes and submits their own copy, in their own fork.

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in English.

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 498 | 600 | 93.5% | 100% |
| t0-small | 50 | 121 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

> The large model buys accuracy (93.5% vs 72.5% for `t0-small`). It costs latency (p50 498 ms vs 50 ms) and money, since every review is a large call.

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 34.8 | 250 | 460 | 26 | 0 | 93.7% |
| t1-after | 9.9 | 402 | 1021 | 0 | 23 | 93.2% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

> The 26 failed requests are counted as errors, not as slow requests, so p50 only measures the fast ones. The **errors** column and the **run time** (34.8 s vs 9.9 s) show the real problem.

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

> A hung request keeps its connection and worker busy the whole time it waits. When enough of them are stuck, other requests have to wait too, so every request gets slow.

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

> Save them and score them later with a background job. This is the **asynchronous / batch** topology (T4).

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

> At 0.1 s, almost every request times out, because a normal call takes about 0.5 s. At 10 s, hung calls hold their connections too long, and the system gets stuck almost like before.

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 16 | 511 | 91.5% | 38% |
| t2-0.7 | 0.7 | 384 | 567 | 93.0% | 53% |
| t2-0.8 | 0.8 | 434 | 560 | 93.5% | 64% |
| t2-0.9 | 0.9 | 469 | 596 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

> Threshold **0.8**: it gets the same accuracy as the large model (93.5%) with 36% fewer large calls. I give up some **latency** (p50 = 434 ms).

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

> 64% of requests still go to the large model, so p50 and p95 both land on the slow path. The slowest requests run both models, so p95 can't get better.

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

> The small model can be confident and wrong on sarcasm (e.g., "Great, only waited an hour!"). The owner then misses a bad review.

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 17 | 535 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

> Short, repeated reviews like "Delicious!" benefit. Long, unique reviews never hit the cache.

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

> The cache must be cleared, or it will keep serving the old model's answers. Put the model version in the cache key.

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

> The cache cut large calls from 53% to 38% with no accuracy loss. 
