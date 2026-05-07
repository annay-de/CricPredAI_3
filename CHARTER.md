# Project Charter — ECO 6810 Final Project

---

## Header

| Field | Value |
|---|---|
| Team members | Annay De, Tanmay Singh, Siddhant Mukherjee |
| Project type | Predictive |
| Estimated hours per person | 10-12 |
| Charter version | v2 |
| Date | 2026-05-07 |

---

## 1. Problem and Stakeholder

The stakeholder is an **IPL franchise strategy and analytics unit** — specifically, the team analyst responsible for playing-XI selection before each match. This is an economics-of-decision problem: the analyst must allocate a fixed squad budget (11 slots) across batters, bowlers, and all-rounders under uncertainty about how individual players will perform against a specific opponent at a specific venue. The decision moment is **team selection, approximately 24 hours before the match**, when the venue, pitch report, weather forecast, and opposing squad are known.

The tool this project builds allows the analyst to simulate ball-by-ball match outcomes under different XI configurations and match conditions. This directly informs the resource-allocation decision: which combination of players maximises the franchise's expected score or win probability given the context? The framing is identical in structure to the economic decision problems this course addresses — constrained optimisation under uncertainty, with a probabilistic model replacing the analyst's intuition.

---

## 2. Main Outcome Variable

- **Name:** Multiclass log-loss on ball-outcome prediction
- **Unit:** Nats (log-loss, lower is better)
- **Source table / column:** IPL ball-by-ball dataset (`chaitu20/ipl-dataset2008-2025` on Kaggle); the target column is the delivery outcome class (dot, 1, 2, 3, 4, 6, W, WD, NB, LB, B)
- **Population / panel:** All legal and extra deliveries across IPL seasons 2008–2025, split into training rows and a held-out test slice (final 15% of matches by date)

Log-loss is the primary metric because this is a probabilistic simulation project. A model that produces better-calibrated probability distributions over ball outcomes generates more realistic simulations and is therefore more useful to the franchise analyst than one that only maximises hard-classification accuracy.

---

## 3. Main Quantitative Success Threshold

**Predictive threshold:** Out-of-sample multiclass log-loss of the primary model on the held-out test split is strictly lower than the empirical baseline log-loss on the same split.

In concrete form: `primary_log_loss < baseline_log_loss`, where both values are written to `outputs/primary_metric.json` and `outputs/baseline_metric.json` respectively by `uv run main.py`.

The current run already satisfies this — the primary model beats the empirical baseline — and the threshold requires that this holds on the final submission run.

---

## 4. Baseline to Beat

The baseline is an **empirical prior model**: for each possible ball outcome, it estimates the probability directly from the marginal frequency of that outcome in the training data, ignoring all match-state features. This is equivalent to a naive "historical average" forecast.

This baseline is computed before any ML model is trained, using only the training split. Its log-loss is recorded in `outputs/baseline_metric.json`. Any model that uses match-state features (over, wickets, batter, bowler, venue, etc.) should in principle beat this baseline, and the project's success threshold requires that it does.

---

## 5. Falsifiable Hypothesis

A Random Forest model trained on delivery-level match-state features (over, wickets fallen, batter identity, bowler identity, venue, innings phase) will achieve strictly lower multiclass log-loss than the empirical marginal-frequency baseline on held-out IPL deliveries from seasons not seen during training.

---

## 6. Data Sources and Access Plan

**Source:** IPL ball-by-ball dataset, 2008–2025  
**Provider:** Kaggle — dataset identifier `chaitu20/ipl-dataset2008-2025`  
**URL:** https://www.kaggle.com/datasets/chaitu20/ipl-dataset2008-2025  
**Licence:** Public community dataset on Kaggle (free to use for educational and research purposes)  
**Access method:** Programmatic download via `kagglehub` Python library

**10-line data probe** (also saved as `Data/data_probe.py`):

```python
import kagglehub
from kagglehub import KaggleDatasetAdapter

df = kagglehub.load_dataset(
    KaggleDatasetAdapter.PANDAS,
    "chaitu20/ipl-dataset2008-2025",
    "",
)
print(f"Rows: {len(df)}")
print(f"Columns: {list(df.columns)}")
print(df.head(3))
```

Run with: `pip install kagglehub[pandas-datasets] && python Data/data_probe.py`

The deployed Streamlit app and `uv run main.py` do **not** require the raw CSV at runtime — they load saved artifacts from `artifacts/`. The probe above exists purely to demonstrate reproducible data access for grading purposes.

---

## 7. Scope Limits

- We are **not** estimating a causal effect of any player, policy, or intervention. This is a predictive project.
- We are **not** predicting match winners directly. The output is a probability distribution over ball outcomes, from which match scores are simulated.
- We are **not** harmonising venue names across all historical seasons; known venue name variants are mapped but residual mismatches may exist.
- We are **not** building player injury or availability forecasting.
- We are **not** shipping a production-grade mobile application.
- Secondary metrics (accuracy, balanced accuracy, macro-F1) may be reported but are **not** the graded success criterion.
- Weather and pitch inputs are user-specified scenario assumptions, not data-trained variables.

---

## 8. Risks and Fallback

**Risk:** The Kaggle dataset API changes structure or requires authentication that is unavailable in the grading environment.

**Fallback:** The repo includes `artifacts/` with all pre-trained model files and metadata. `uv run main.py` runs end-to-end using these saved artifacts and does not require a live Kaggle download. The grader can verify data access separately by running `Data/data_probe.py` in any environment with a Kaggle account. If the probe also fails, the `artifacts/metadata.json` file documents the training data shape (row count, column names, season range) as a static record of what was used.

---

## 9. Reproducibility Checklist

- [x] `uv run main.py` runs end-to-end with no manual intervention and completes in under 10 minutes
- [x] It writes `outputs/primary_metric.json` with shape `{"metric_name": "log_loss", "value": <number>, "threshold": <baseline_value>, "passed": true}`
- [x] It writes `outputs/baseline_metric.json` with shape `{"metric_name": "log_loss_baseline", "value": <number>}`
- [x] It writes `outputs/milestone_manifest.json` documenting run status
- [x] `README.md` documents the run command and expected outputs in under 20 lines
- [x] All data sources are either fetched in-script or accessible via the documented Kaggle probe
- [x] Saved model artifacts in `artifacts/` allow the app and main.py to run without the raw CSV

---
*Submitted by: Annay De, Tanmay Singh, Siddhant Mukherjee*
