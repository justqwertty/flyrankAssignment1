# Which content should a reviewer refresh first?

- **Author:** Omar Hisham
- **Lane:** Content opportunity scoring / refresh-priority ranking
- **Date:** August 2026
- **Live paper:** https://justqwertty.github.io/flyrankAssignment1/

## Abstract

FlyRank’s content-review problem is a simple operational constraint: a reviewer cannot inspect every existing item in a cycle, so which ones should come first? I evaluated a transparent weighted rule and three learned classifiers on an anonymized 30,000-item FlyRank starter release, then used their scores only to rank a review queue. The primary comparison uses a client-held-out test set and Precision@50, so every model and the baseline are judged on the same unseen items. The selected random forest measured Precision@50 of 0.74 versus 0.24 for the rule baseline, while the held-out declining-label base rate was 0.54. This is decision-support evidence for helping a FlyRank content reviewer triage an auditable shortlist against this proxy label—not evidence that a refresh causes traffic recovery or that the model predicts a search engine's ranking system.

## Introduction / problem statement

This FlyRank content-operations case study addresses the decision of which items receive one of roughly 50 manual review slots. The queue is a reviewer aid, never an automatic publishing decision; false positives waste editorial time and false negatives may leave a potential decline unnoticed. Precision@50 is the primary metric because it measures the quality of the finite shortlist.

## Data

The analysis uses the bundled anonymized FlyRank ML Internship starter release: 30,000 pseudonymized content items from 32 pseudonymized clients. Each row has trailing-90-day aggregates and embedded recent/previous 30-day comparison windows. I excluded identifiers as features, and excluded titles, domains, URLs, keywords, raw queries, provider/model fields, product flags, `trend_direction`, `trend_pct`, and all current-window outcome columns. The latter are label-derived or outcome-period fields and would leak. The full warehouse (~79M daily rows) is not used for the paper's reported aggregates.

## Methodology

The proxy target is `is_declining_label = 1` when `trend_direction == "down"`, a decline of more than 20% between the embedded 30-day windows. This is a rule-defined contemporaneous proxy, not a future observed outcome. Eighteen numeric and eight categorical/bucketed observable signals become 52 features after encoding. The baseline weights visibility (40%), freshness risk (30%), position opportunity (25%), and content-depth gap (5%). A seed-42 client-held-out split keeps whole pseudonymized clients out of test (27,675 train / 2,325 test); every comparison uses that same test partition. Direction, trend percentage, and last-30-day outcome fields were excluded, and a deliberate leak harness was used to ensure the audit detects suspiciously perfect scores.

## Results

| Approach | ROC AUC | Average precision | Precision@50 | F1 |
|---|---:|---:|---:|---:|
| Rule baseline | 0.627 | 0.468 | 0.240 | 0.274 |
| Logistic regression | 0.700 | 0.522 | 0.400 | 0.566 |
| Decision tree | 0.742 | 0.575 | 0.620 | 0.634 |
| Random forest | 0.750 | 0.618 | 0.740 | 0.640 |

The test-set proxy-label base rate is 0.542. The random forest's 0.74 Precision@50 means 37 of the first 50 ranked items carried the proxy label, compared with 12 for the baseline. Its leading feature importances were days with impressions (0.158), log impressions over 90 days (0.129), average position (0.109), and content age (0.095). These are model-use associations, not causal effects.

## Limitations & honest framing

The label is derived from historical windows and does not prove future decline, refresh value, or recovery. One fixed split and one seed do not quantify uncertainty, so repeated grouped or time-forward validation is needed. Results from this 30,000-row teaching slice do not automatically generalize to the full warehouse, new clients, or a changing search environment. Human review must assess brand, accuracy, seasonality, compliance, and editorial quality.

## Ranked recommendations

1. Review the top 50 random-forest candidates first, with a human confirming each action.
2. Use reason codes to route items to refresh, CTR review, engagement review, or monitoring; never automatically change content.
3. Retain the transparent baseline as an inspectable control queue.
4. Run time-forward, repeated-client validation before any production rollout.
5. Monitor base rate, Precision@50, and feature drift; retire the ranking if it no longer beats the baseline.

## Reproducibility

Install `requirements.txt` and run `python scripts/run_all.py`; this recreates the aggregate model results and charts from the bundled anonymized slice. The supporting notebooks live in `work/notebooks/`, including `capstone.ipynb`. The model seed is 42.

## Acknowledgments & data credit

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
