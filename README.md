# Evaluating a Self-Hosted Multi-Agent System on Real Operational Data

A methodology and results writeup from a production wet-pilot of a multi-agent
decision-support system, run end-to-end on a self-hosted open-weight model with
zero hot-path API spend.

This document describes how the system was tested, what was measured, and what
the numbers were, including the hypotheses that failed. The goal is to show the
evaluation harness and the evidence discipline behind it, not to market a result.

---

## What was evaluated

The system places a federation of role-specialized agents next to a mid-market
construction contractor's live operations data. For this pilot the target task
was a hard, real one: given the operational record, pick the right field lead
for a job, the way a senior operator would, and justify the pick from evidence
the system can actually point to.

The whole pilot ran on a self-hosted open-weight model (NousResearch
Hermes-4-70B-FP8) served with vLLM on a single rented H100. No frontier-model
API was in the decision hot path. The only paid API call anywhere in the run was
an offline narrative-translation step that does not influence decisions.

## Why the methodology is the interesting part

It is easy to demo an agent that looks smart. It is hard to know whether it is
right, and harder to know whether your measurement of "right" is trustworthy.
The harness was built to answer both.

### Two-database holdout

The agent reasons over one database (its worldview). Ground truth lives in a
second database the agent never queries. The holdout window is everything that
happened between the worldview database's last timestamp and the present, so the
prediction targets are real operational decisions the agent could not have seen.
This keeps the answer key out of the agent's reach by construction, not by
convention.

### Pre-registered hypotheses

Every claim was written down as a hypothesis with an explicit numeric threshold
before the run, then graded pass or fail against that threshold afterward. This
is the part that keeps an evaluation honest: you cannot move the goalposts after
you see the data if the goalposts were committed in advance.

### Reproducible scoring

Each decision was scored on a multi-axis rubric by independent raters. Inter-rater
reproducibility was Cohen's kappa = 0.95, so the scores reflect the rubric rather
than any single grader's taste. A separate first-person judgment pass re-evaluated
every retained pick against an operations-defensibility standard.

### Cost and reliability budget

The run had a pre-set dollar budget and a compose-reliability target, both treated
as first-class pass/fail gates alongside capability.

### Regression gate

The harness is wired into CI: a build fails on a 10-point capability drop or a 10%
p95 latency regression, so capability cannot silently rot between changes.

---

## Headline results

On a fixture where the operationally-correct answer is provably derivable from the
substrate:

| Measure | Result |
|---|---|
| Strict capability (exact operator-match) | **66.7%** (10/15), pre-registered threshold was 55% |
| Operations-defensible (independent judgment pass) | **86.7%** one-pass approve |
| Substrate-consistent rationales | **100%** |
| Hallucinated entities | **0** |
| Pathway-citation rate on correct picks | **100%** |
| Inter-rater reproducibility | **kappa = 0.95** |
| Fabrication-guard trigger rate | **0%** (0 of 51 held-out decisions) |
| Compose reliability | **100%** (279 of 279) |
| Decision-path API spend | **$0.00** |
| Total compute (including debugging) | **~$8.50** against a $26.66 budget |

The agent cited the specific evidence behind each pick when that evidence was
present, and explicitly noted its absence when it was not, which is the kind of
calibrated self-assessment you want before you trust a recommendation.

---

## The hypotheses that failed, and why that matters

Of the pre-registered hypotheses, several failed. Reporting them is the point.

- A "divergence from a control arm" metric came back at 0% and was marked
  **instrument-invalidated**. Investigation showed the fixture's feedback hints
  had been authored to align with the control's default picks, so both arms
  reached the same answer for the right reason on each side. The metric was
  measuring the fixture, not the model. This was traced, documented, and given a
  surgical fix for the next run rather than quietly dropped.
- An autonomous tool-call-rate hypothesis failed as anticipated, and the
  corresponding claim was reframed: the system is prompt-grounded executable
  retrieval with a human approving each proposal, not an autonomous command
  loop. That is a deliberate architectural posture, and the evaluation made it
  explicit instead of papering over it.

A pre-registered evaluation where some hypotheses fail and the failures are
explained is more credible than one where everything passes. The capability
gates that did pass were the ones the harness was designed to measure cleanly,
and they were reproducible.

---

## What this demonstrates

- A multi-agent system can produce operations-grade decisions on real data
  while running entirely on a self-hosted open-weight model, with no operational
  data and no decision-path spend leaving the owner's environment.
- The evidence for that claim rests on a holdout design that withholds the answer
  key, pre-registered thresholds, reproducible multi-rater scoring, and an honest
  accounting of what did not work.

## What this is not

This was a proof-of-concept on a single fixture, not a production benchmark.
The capability number is reported for one task on one cohort, the sample is small
(15 eligible decisions), and the next run was scoped specifically to close the
instrument gaps found here. The numbers above are what they are: early, honest,
and reproducible.

---

*Client identifiers, names, internal system names, and infrastructure details
have been removed. Raw run artifacts and the full pre-registration live in a
private repository.*
