# Capstone Report — Refresh & Content Opportunity Scoring

- *Author:* Shoaib
- *Lane:* Refresh / Content Opportunity Scoring
- *Repo:* https://github.com/Shoaib585/flyrank-ml-internship
- *Date:* September 2026

## 0. Abstract
How can we predict which declining web pages require immediate content refreshing to recover organic search traffic? Utilizing the FlyRank warehouse dataset from March 2026, we extracted historical click logs, impression trends, and days-since-update features. We implemented a machine learning ranking model evaluated against a transparent staleness rule baseline. The model successfully identified decaying pages with a measurable lift over the baseline rules. The output is a prioritized daily action queue designed for content editors to optimize stale articles.

## 1. Problem framing
This work supports the decision of content editors and SEO strategists to choose which articles to refresh first. 
- **Unit of analysis:** A single URL performance metric on a specific day.
- **Output:** A ranked action score and a reason code (`STALE_HIGH_THROUGHPUT_DECAY`).
- **Human Action:** An editor schedules a content rewrite or update for top-ranked pages.
- **Cost of a wrong call:** Wasting editorial resources updating an article that has plateaued naturally, versus missing a decaying page that loses organic rank. 
Data/ML helps because manual review cannot scale across thousands of warehouse URLs daily.

## 2. Data safety
We used the FlyRank warehouse `search_console_url` release via DuckDB. We deliberately excluded raw client IDs, private query texts, and future performance windows to prevent leakage. Label-derived fields like future clicks were strictly separated from training features. We confirm zero client-identifying or sensitive information exists anywhere in `work/`.

## 3. Baseline
The baseline is a transparent heuristic rule: if an article's days since last update exceeds 150 days and recent clicks are above 50, flag it with a rule-based priority score. It serves as a fair, interpretable comparison against our ML model on the exact same dataset and evaluation metrics.

## 4. Model / analysis
We applied a scikit-learn regression/ranking model fitting historical staleness, average search position, and recent clicks. 
- **Target definition:** Predicting organic traffic preservation or decay risk for the upcoming window.
- **Features used:** `days_since_update`, `avg_position`, `recent_clicks`. 
- **Features excluded:** Future window metrics and label-derived data.

## 5. Evaluation
We performed a time-aware split on the panel data. The evaluation compares model precision and ranking metrics against the baseline rule. Error analysis reveals that the model occasionally over-scores seasonal query drops where staleness is not the root cause.

## 6. Interpretation
The analysis confirms that content staleness combined with high historical impressions is a strong signal for content decay. Negative results show that pages with very low initial volume do not benefit from standard updates, confirming that traffic thresholds must gate refresh recommendations.

## 7. Recommendation
The output provides a ranked action queue of pages needing content review. FlyRank editors should review the top decile daily. Confidence is decision-support level (directional and observed); recommendations assume stable search intent.

## 8. Reproducibility
To re-run from a fresh clone:
1. Install dependencies: `pip install pandas pyarrow duckdb scikit-learn huggingface_hub`
2. Set your Hugging Face `HF_TOKEN` secret.
3. Run `work/notebooks/w03_data_contract.ipynb` and `work/notebooks/w04_baseline_score.ipynb`.
Random seed is set to `42` across all scripts for deterministic splits.

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset. Learn more at [FlyRank](https://flyrank.ai).
