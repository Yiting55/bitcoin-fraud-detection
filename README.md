# Fraudulent Bitcoin Transaction Detection

This project predicts the probability that a Bitcoin transaction is illicit.
Transactions are graph nodes, and directed edges represent payment flows between
transactions. The challenge combines anonymous numeric features, partially known
labels, class imbalance and prediction on future time periods.

The complete development workflow and final model are implemented in
**[e1155697.ipynb](e1155697.ipynb)**. The notebook is self-contained: no additional
project scripts, saved models or historical result files are required.

## Repository contents

```text
<repository-root>/
├── e1155697.ipynb       # End-to-end experiments and final submission generation
├── requirements.txt    # Pinned Python dependencies
├── README.md
└── data/
    └── README.md       # Instructions for obtaining and placing the dataset
```

## Dataset

The raw dataset is not included in this repository. Download and extract the
files from the Kaggle competition `bt-4012-competition-2026` after accepting its
rules. Place all four files directly in `data/`:

```text
data/train.csv
data/test.csv
data/txs_edgelist.csv
data/sample_submission.csv
```

See [data/README.md](data/README.md) for file descriptions. The actual numeric
feature columns are named `feat_1` through `feat_165`.

**Launch Jupyter from the repository root.** The notebook uses
`DATA = Path.cwd() / 'data'`, so it reads these files using a relative path.
Do not upload the raw CSV files or Kaggle credentials to GitHub.

## Installation and execution

The notebook was verified with **Python 3.11.9** and the dependencies pinned in
`requirements.txt`. From the repository root, on macOS or Linux:

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter notebook e1155697.ipynb
```

Select the Python environment containing those dependencies and run all cells
from top to bottom. The notebook uses two CPU threads and does not require a
GPU. A complete verified local run took approximately 13 minutes; runtime varies
with hardware. Every execution creates a new output directory.

## Development workflow

The notebook includes both the reasoning and executable comparisons behind the
final model:

1. Audit labels, missing/nonfinite values, transaction IDs, class imbalance and
   temporal distributions. Exclude unknown labels from supervised fitting.
2. Examine anonymous-feature correlations and explain why near-perfect displayed
   correlations do not automatically justify removing features.
3. Establish logistic regression, random forest and initial HGB baselines, then
   compare two HGB regularization settings.
4. Compare in-degree, predecessor aggregates, two-hop counts and transaction-minus-
   predecessor differences; retain only in-degree for the final HGB.
5. Compare fixed HGB, XGBoost, LightGBM, Plain CatBoost and Ordered CatBoost
   recipes, individual models and equal-weight blends on the same temporal windows.
6. Evaluate a compact MLP using training-period-only epoch selection, then examine
   probability weights and three-seed CatBoost averaging.
7. Fit the selected models on all known training labels, predict the test set,
   validate the submission format and export both selected weight combinations.

The initial logistic baseline trains on steps 1–25 and validates on 26–29,
producing local ROC AUC **0.93117**. The longer comparisons train on
1–12, 1–15, 1–18 and 1–21, each validating on the following 14 steps.
Three-seed and three-seed weight diagnostics are shown separately for the latest
origin, 1–21 → 22–35.

These validation periods were reused during development. Local metrics are not
untouched holdout estimates and are distinct from competition scores. Final
weight selection also used competition feedback; the notebook does not claim
that the final blend is optimal in every local window.

## Final models and selected submissions

Both models train on labeled transactions from steps **1–35** and predict test
transactions from steps **36–49**. Unknown labels are not treated as legal
transactions. Transaction ID, time step and label are not input features.

| Component | Inputs | Fixed settings |
|---|---|---|
| HGB | 165 original features + in-degree | 250 iterations, learning rate 0.05, 31 leaves, minimum 20 samples/leaf, L2=5, seed 42, early stopping off |
| Ordered CatBoost | 165 original features | 700 iterations, learning rate 0.04, depth 6, L2=8, seeds 42/43/44 |

CatBoost probabilities are averaged across the three seeds, then combined with
HGB probabilities using the weights below. The final models use probability
averaging, without rank transformation, graph smoothing or neural stacking.

| Generated filename | CatBoost mean weight | HGB weight | Recorded online ROC AUC |
|---|---:|---:|---:|
| `submission_cb75_hgb25.csv` | 75% | 25% | **0.95436** |
| `submission_cb50_hgb50.csv` | 50% | 50% | **0.95224** |

The online scores are recorded competition results. They cannot be recalculated
locally because test labels are unavailable; the notebook calculates development
metrics separately.

## Generated outputs

Each run saves models, comparison tables and predictions under:

```text
outputs/notebook_e1155697/<timestamp>_<run-id>/
└── submission/
    ├── submission_cb75_hgb25.csv
    ├── submission_cb50_hgb50.csv
    ├── prediction_components.csv
    └── verification.json
```

Both submission CSVs have columns `index,target` and 15,329 rows. The notebook
checks index order, uniqueness, missing values and score bounds, and compares
file checksums with the selected original submissions. In the verified local
run, both generated CSVs matched the selected originals byte for byte.

Generated files are outputs, not prerequisites. Nothing is automatically uploaded
to Kaggle or GitHub. If adding data or running the notebook inside a Git checkout,
use a `.gitignore` to exclude raw data, `.venv/`, `outputs/` and notebook checkpoints.
