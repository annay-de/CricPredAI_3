# CricPredAI V3.0

A ball-by-ball IPL decision-support simulator.

## What changed in V3.0

- Removed the unreliable preset team-pool dependency from the main workflow.
- User types team names manually.
- Player selection uses autocomplete-style Streamlit multiselects based on dataset-derived player names.
- Adds venue, pitch, weather, toss winner and toss decision.
- Adds Random Forest and XGBoost training support, alongside logistic, Extra Trees, HistGradientBoosting and baseline prior.
- Uses empirical calibration/blending in simulation so ML probabilities do not create absurd collapses.
- Correctly handles wides/no-balls at the start of an over: bowler is locked until six legal balls are bowled.
- Adds no-balls, free-hit logic, wides, byes and leg-byes using dataset-derived extra distributions.
- Removes broken benchmark simulation mode.
- Combines both innings scorecards on one page.
- Superimposes score progression curves for both innings and marks wickets.
- Adds ball-by-ball verification table and optional commentary.

## Deploying

Upload the extracted folder to GitHub and deploy `app.py` on Streamlit Cloud.

The deployed app does not need `IPL.csv` because it loads saved artefacts from `artifacts/`.

## Retraining

To retrain, place `IPL.csv` in this folder and run:

```bash
python train_models.py
```

This will regenerate the files under `artifacts/`.
