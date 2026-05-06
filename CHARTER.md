# CricPredAI Charter

## Project title

CricPredAI: A probabilistic ball-by-ball IPL match simulator and decision-support system

## Project summary

CricPredAI is a cricket analytics project that uses historical IPL ball-by-ball data to simulate T20 matches at the delivery level. The system estimates probabilities for ball outcomes such as dots, singles, boundaries, wickets, wides, no-balls, byes and leg-byes, and then uses these probabilities to generate full innings and match simulations.

The project is designed as a decision-support and experimentation tool rather than a simple final-score predictor. Users can define two teams, select playing XIs from dataset-derived player lists, choose match conditions, select a model, and then simulate a match with detailed scorecards, ball-by-ball verification, progression graphs, and simulation distributions.

## Research and product question

Can historical IPL ball-by-ball data be used to build a calibrated probabilistic simulator that produces realistic T20 match outcomes and supports tactical experimentation over team composition, batting order, bowling choices, venue conditions and match state?

## Main data source

The main data source is an IPL ball-by-ball dataset stored locally as `IPL.csv`.

The dataset is used for:
- delivery-level match outcomes,
- batter and bowler identities,
- batting and bowling teams,
- innings state,
- venue information,
- wickets,
- extras,
- toss information,
- and match progression.

The deployed Streamlit app does not require the raw `IPL.csv` file at runtime. Instead, the training pipeline generates saved artefacts under `artifacts/`, which are then loaded by the app.

## Current repository structure

The current V3 repository contains the following main components:

- `app.py`: Streamlit frontend for the simulator
- `simulator.py`: match simulation engine
- `train_models.py`: training and artefact generation pipeline
- `model_wrappers.py`: wrapper support for XGBoost model output
- `requirements.txt`: Python dependencies
- `artifacts/`: saved model artefacts, metadata and model reports

The current frontend allows users to:
- enter custom team names,
- select playing XIs using dataset-derived player lists,
- choose venue, pitch, weather, toss winner and toss decision,
- select a model,
- run match simulations,
- inspect scorecards,
- inspect ball-by-ball outputs,
- view rule checks,
- and compare simulated score distributions.

## Baseline model

The baseline model is an empirical prior model based on historical IPL delivery outcome frequencies. It estimates the probability of each ball outcome directly from the training data.

This is used as a benchmark and also as a stabilising reference distribution for simulation. It is especially useful because historical outcome frequencies are naturally calibrated to real IPL scoring patterns.

## Primary modelling approach

The primary modelling approach is multiclass delivery-level outcome prediction. Each ball is treated as an observation, and the model predicts the probability distribution across possible outcomes.

Current models include:
- empirical baseline prior,
- Random Forest,
- XGBoost support,
- and calibrated/blended simulation logic.

The simulator uses model-predicted probabilities along with empirical calibration/blending so that raw machine learning probabilities do not produce unrealistic match paths such as excessive collapses or implausible scoring patterns.

## Outcome classes

The current outcome classes include:

- `0`: dot ball
- `1`: one run
- `2`: two runs
- `3`: three runs
- `4`: four runs
- `6`: six runs
- `W`: wicket
- `WD`: wide
- `NB`: no-ball
- `LB`: leg-bye
- `B`: bye

## Features currently used

The current training pipeline uses a mixture of numerical and categorical match-state features.

Numerical features include:
- innings,
- over,
- legal ball number,
- team runs,
- team wickets,
- batter runs,
- batter balls.

Categorical features include:
- phase of innings,
- batting team,
- bowling team,
- batter,
- bowler,
- venue,
- toss decision.

## Evaluation metrics

The primary metric is:

- multiclass log loss.

Additional model diagnostics include:
- accuracy,
- balanced accuracy,
- macro-F1 score,
- model comparison table.

Log loss is the main metric because this is a probabilistic simulation project. A model that produces better calibrated probability distributions is more useful than a model that only maximises hard classification accuracy.

## Current milestone outputs

For the milestone, the repository should include or generate:

- `outputs/baseline_metric.json`
- `outputs/primary_metric.json`
- `outputs/milestone_manifest.json`
- `artifacts/model_report.csv`
- `artifacts/metadata.json`

The milestone runner should be executable from the repository root using:

```bash
uv run main.py
```

The Streamlit app remains separately executable using:

```bash
streamlit run app.py
```

## Current implementation status

Implemented:
- Streamlit app interface
- custom team names
- playing XI selection
- venue, pitch, weather, toss and model controls
- two-innings simulation
- target chase logic
- batting scorecards
- bowling scorecards
- fall of wickets
- ball-by-ball verification table
- optional commentary-style output
- score progression graph
- simulation distribution output
- saved model artefact loading

Partially implemented:
- empirical calibration/blending
- extra outcome handling
- no-ball and free-hit logic
- model diagnostics
- rule checking

Still under improvement:
- stronger player-level feature engineering,
- venue name standardisation,
- richer use of matchup history,
- improved probability calibration,
- more rigorous season-wise validation,
- better use of the full dataset richness,
- and more stable ML performance relative to the empirical baseline.

## Known limitations

The current V3 version is a working prototype, but the modelling layer is not yet the final intended version.

Known limitations:
- venue names are not yet fully standardised,
- weather and pitch are mostly scenario assumptions rather than deeply data-trained variables,
- player strength is not yet captured as strongly as desired,
- matchup-specific features are limited,
- some ML models may be weaker than the empirical baseline,
- probability calibration needs further work,
- and the raw dataset is not used at runtime because the app relies on saved artefacts.

These limitations are explicitly part of the next development stage.

## Planned improvements

The next modelling-first version will focus on:

1. Cleaning and standardising stadium names.
2. Standardising player names.
3. Creating stronger batter features such as strike rate, boundary rate, dismissal rate and phase-wise scoring.
4. Creating stronger bowler features such as economy, wicket rate, dot-ball rate, phase-wise performance and death-over skill.
5. Adding matchup-level features where sample size permits.
6. Training Random Forest and XGBoost models more rigorously.
7. Calibrating predicted probabilities using proper calibration methods.
8. Validating model performance through time-based or season-based splits.
9. Comparing simulated score distributions against actual IPL score distributions.
10. Reducing reliance on empirical baseline blending as model quality improves.

## Expected final deliverable

The final deliverable will be an interactive probabilistic cricket simulator that can:

- simulate IPL-style T20 matches ball by ball,
- allow user-defined team and player selection,
- generate realistic scorecards and match paths,
- show uncertainty through repeated simulations,
- explain model and simulation outputs,
- and support tactical cricket decision analysis.

The intended final positioning is not merely a cricket score predictor, but a calibrated cricket decision-support simulator for analysing how team composition, batting order, bowling choices and match conditions affect expected match outcomes.
