# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Adarsh24BDA70227
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** `Adarsh24BDA70227/flyrank-ml-track`
- **Date:** 2026-08-30

## 0. Abstract

We investigate whether observed search-performance and content signals can prioritize pages for refresh review. Using the bundled 30,000-row anonymized FlyRank content-refresh dataset, we prepare page-level features while excluding identifiers and label-derived trend fields from the model. We compare a transparent baseline with Logistic Regression, Decision Tree, and Random Forest using a client-holdout evaluation design. Random Forest is the selected model by Precision@50, reaching 0.740 versus 0.240 for the baseline, with ROC-AUC 0.750 and average precision 0.618 on the held-out split. The resulting ranked queue is a decision-support aid for human content review rather than evidence about Google’s causal ranking algorithm.

## 1. Problem framing

The practical decision is **which pages should a reviewer inspect first** when editorial capacity is limited. The unit of analysis is an anonymized content page. The output is an opportunity score, rank, reason codes, and suggested action. A wrong positive call costs reviewer time; a wrong negative call may delay useful content maintenance. Ranking the highest-priority candidates first is therefore more useful than optimizing a generic classification accuracy alone.

## 2. Data safety

The project uses the bundled anonymized starter dataset at `data/raw/content_refresh_anonymized.csv` with 30,000 rows and 44 columns before preparation. The analysis uses page-level search, content, and engagement signals. No client names, domains, URLs, titles, keywords, credentials, or private queries are included. `client_id` is used only to construct a grouped holdout split and is never used as a model feature.

Rows with no 90-day impressions and pages younger than 90 days are excluded, followed by de-duplication on `content_id`. The positive label is the observed proxy `trend_direction == "down"`; this is explicitly treated as a current-window decline proxy, not a future-window forecast. `trend_direction` and `trend_pct` are excluded from the feature matrix because they are label-derived.

## 3. Baseline

The baseline is a transparent weighted score combining visibility, freshness risk, position opportunity, and content-depth gap. On the same held-out evaluation set, its ROC-AUC is 0.627, average precision is 0.468, and Precision@50 is 0.240. This establishes a simple, interpretable reference point for the learned models.

## 4. Model / analysis

The feature vector contains 52 encoded features derived from safe numeric and categorical fields. Numeric signals include search volume, competition, CPC, content length, log-transformed 90-day impressions/clicks/sessions, availability days, content age, days since last update, CTR, average position, engagement rate, scroll rate, and AI-traffic percentage. Categorical features include competition level, content type, intent, age, freshness, word-count, impression, and position tiers.

Three learned models are compared: Logistic Regression, Decision Tree, and Random Forest. Random Forest is selected using Precision@50 because the operational task is top-of-queue prioritization.

## 5. Evaluation

The validation strategy is `client_holdout`: 20% of clients are held out when enough clients and both target classes are available. The resulting evaluation set contains 2,325 rows; training contains 27,675 rows.

| Model | ROC-AUC | Average Precision | Precision@50 | Recall | F1 |
|---|---:|---:|---:|---:|---:|
| Baseline rules | 0.627 | 0.468 | 0.240 | 0.189 | 0.274 |
| Logistic Regression | 0.700 | 0.522 | 0.400 | 0.567 | 0.566 |
| Decision Tree | 0.742 | 0.575 | 0.580 | 0.716 | 0.634 |
| **Random Forest** | **0.750** | **0.618** | **0.740** | **0.744** | **0.640** |

The overall positive-label rate is 0.542. The Random Forest therefore concentrates positive proxy cases substantially more effectively in the top 50 than the baseline.

## 6. Interpretation

The strongest Random Forest feature importances are `days_with_impressions` (0.158), `log_impressions_90d` (0.129), `avg_position` (0.109), and `content_age_days` (0.095). These should be interpreted as observed associations within this modeling setup, not as causal ranking factors.

The queue contains 3,605 high-confidence, 11,395 medium-confidence, and 15,000 low-confidence items. Suggested actions include `monitor` (13,093), `refresh` (8,178), `refresh_and_review_ctr` (6,657), `refresh_and_review_engagement` (1,990), and `expand_and_refresh` (82).

## 7. Recommendation

Use the queue as a reviewer aid. Start with high-confidence rows, inspect the reason codes, verify the page manually, and then choose the editorial action. A `refresh_and_review_ctr` recommendation indicates a page with visible demand and a CTR-review signal; `refresh_and_review_engagement` indicates an engagement-review signal; `expand_and_refresh` flags a visible page with a content-depth opportunity; `monitor` is appropriate when the evidence is weaker.

## 8. Reproducibility

The reproducible pipeline is in `scripts/01_prepare_features.py` through `scripts/04_evaluate_and_export.py`. The capstone notebook is `work/notebooks/capstone.ipynb`. Generated charts are in `outputs/charts/`. The deployed paper is in `paper/index.html`.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. Data source: [FlyRank](https://flyrank.ai).

## Honest framing

This work measures patterns in the supplied dataset and converts them into a prioritization score. It does **not** prove Google ranking factors, causal refresh effects, or guaranteed traffic gains. Recommendations are directional and intended to support human editorial decisions.
