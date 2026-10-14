# IEEE-CIS Fraud Detection

A machine-learning project for identifying potentially fraudulent transactions using the [IEEE-CIS Fraud Detection competition dataset on Kaggle](https://www.kaggle.com/competitions/ieee-fraud-detection).

The project explores transaction patterns, builds Logistic Regression baselines, and compares XGBoost models using a chronological train/validation/test split. Because fraud is relatively rare in the dataset, the evaluation emphasizes ROC-AUC and average precision (reported as PR-AUC), alongside precision, recall, and F1 at a selected decision threshold.

## Project structure

```text
fraud-detection/
├── notebooks/
│   ├── 01_exploration.ipynb
│   └── 02_xgboost_model.ipynb
├── .gitignore
├── requirements.txt
└── README.md
```

## Dataset

This project uses the IEEE-CIS Fraud Detection competition data. The main files used are:

- `train_transaction.csv` — transaction-level features and the `isFraud` target.
- `train_identity.csv` — identity/device-related features for a subset of transactions.

**Access requirements:** You need a Kaggle account, access to the competition, and acceptance of its rules. The notebooks use `kagglehub` to download the files, so you do not need to manually copy CSVs into a folder.

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/shrayaaa/transaction-fraud-detection.git
cd fraud-detection
```

### 2. Create and activate a virtual environment (optional but recommended)

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Open and run the notebooks

Run the notebooks in order:

1. `notebooks/01_exploration.ipynb` — exploratory data analysis, feature inspection, chronological splitting, and Logistic Regression baselines.
2. `notebooks/02_xgboost_model.ipynb` — XGBoost experiments, feature selection, early stopping, and threshold analysis.

The first dataset setup cell in each notebook calls `kagglehub.login()` and downloads the competition files.

The notebooks may require a machine with sufficient RAM, particularly during XGBoost preprocessing and training. The second notebook also contains optional Google Drive artifact-saving/loading steps used during experimentation; those steps are specific to the original Colab workflow and may need adjustment or skipping when running locally.

## Methodology

### 1. Exploratory analysis and baselines

Notebook 1 examines transaction amounts, product categories, missingness, identity data, and feature groups. It compares Logistic Regression models using progressively richer feature sets, including C/D features and selected V features.

Memory constraint: Training Logistic Regression on the fully expanded feature set was not practical in the available memory environment. The experiments therefore used selected feature groups for the Logistic Regression baselines rather than presenting an all-feature Logistic Regression result.

### 2. Chronological data split

Transactions are ordered by `TransactionDT`, then divided chronologically:

| Split | Approximate share | Rows |
|---|---:|---:|
| Train | 70% | 413,378 |
| Validation | 15% | 88,581 |
| Test | 15% | 88,581 |

This setup evaluates later transaction periods after training on earlier data. The target is imbalanced, with approximately 3.5% fraudulent transactions in each split.

### 3. XGBoost experiments

Notebook 2 compares:
- An initial XGBoost model with selected V features.
- A model using the full V-feature set.
- Reduced feature sets retaining the top 25, 50, or 100 V features, while keeping the other selected features.
- A top-100 V-feature model trained with early stopping against validation AUC-PR.

The decision threshold is selected using validation F1, and a threshold table illustrates the trade-off between precision, recall, missed fraud, and false alarms.

## Results

Metrics below are from the recorded notebook experiments. PR-AUC refers to scikit-learn's `average_precision_score`.

### Logistic Regression baselines

| Experiment | Validation ROC-AUC | Validation PR-AUC |
|---|---:|---:|
| Initial baseline | 0.7889 | 0.1712 |
| Add C/D features | 0.8281 | 0.3062 |
| Add selected V features | 0.8336 | 0.3672 |

### XGBoost model comparison

| Model / feature configuration | Validation ROC-AUC | Validation PR-AUC |
|---|---:|---:|
| Selected V features | 0.9178 | 0.5476 |
| Full V-feature model | 0.9149 | 0.5514 |
| Top 25 V features | 0.9191 | 0.5539 |
| Top 50 V features | 0.9203 | 0.5551 |
| Top 100 V features, early stopping | 0.9195 | 0.5646 |

The top-100 early-stopped model had the strongest validation average precision among these recorded experiments. Its selected validation threshold was approximately **0.783**, chosen to maximize validation F1.

### Final candidate test-period metrics

For the top-100 early-stopped model, the recorded test-period results at the validation-selected threshold were:

| Metric | Result |
|---|---:|
| ROC-AUC | 0.8935 |
| PR-AUC (average precision) | 0.5068 |
| Precision | 0.5870 |
| Recall | 0.4356 |
| F1-score | 0.5001 |

**Evaluation caveat:** The test period was inspected during earlier model comparisons. These results should therefore be treated as a final candidate evaluation, not as an entirely untouched, one-time test estimate. Validation results were used for the final candidate's threshold selection.

## Important considerations

- Feature importance indicates which features the trained model relies on; it does not establish causation.
- The decision threshold is not necessarily optimal for a real fraud operation. The best threshold depends on the relative costs of false positives and missed fraud.
- The dataset is imbalanced, so accuracy alone would be misleading.

## Technologies

- Python
- pandas and NumPy
- scikit-learn
- XGBoost
- SciPy (sparse matrices)
- KaggleHub
- Jupyter Notebook

## License and data usage

The dataset is governed by the terms and rules of the [Kaggle competition](https://www.kaggle.com/competitions/ieee-fraud-detection). Review those terms before using or redistributing the data. This repository does not include the dataset.
