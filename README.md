Predicting NBA Injury Risk from Game-Level Workload Data
=========================================================

OVERVIEW
--------
This project builds a machine learning pipeline to predict NBA player injury
risk using game-level workload data. It covers data collection from the NBA
API and Kaggle, injury data cleaning and filtering, feature engineering
(rolling workload windows, schedule density, injury history), and
classification with XGBoost, Random Forest, Logistic Regression, and
Decision Tree models evaluated with time-series cross-validation.


REQUIREMENTS
------------
  - Python 3.13+
  - uv  (https://docs.astral.sh/uv/)


SETUP
-----

### Install uv

Mac/Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Windows:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```


### Clone the repository

```bash
git clone <repo-url>
cd nba-injury-risk
```

### Install dependencies

```bash
uv sync
```
**Alternative (without uv):**

```bash
pip install -r requirements.txt
```






PROJECT STRUCTURE
-----------------

```
  nba-injury-risk/
  ├── pyproject.toml              Project metadata and dependencies
  ├── uv.lock                     Locked dependency versions
  ├── README.md / README.txt
  ├── data/
  │   ├── raw/                    Raw CSVs from Kaggle and NBA API
  │   │   └── game_logs/          Per-season player game log CSVs
  │   └── processed/              Cleaned and feature-engineered datasets
  └── notebooks/
      ├── 01_data_collection.ipynb
      ├── 02_data_cleaning.ipynb
      ├── 03_feature_engineering.ipynb
      └── 05_modeling.ipynb
```



NOTEBOOKS
---------

* 01_data_collection.ipynb — Data Collection
  * Downloads the NBA Player Injury Stats (1951-2023) dataset from Kaggle via kagglehub, and fetches per-player game logs for the 2015-16 through 2022-23 seasons (regular season and playoffs) from the NBA API. Saves raw CSVs to data/raw/.


* 02_data_cleaning.ipynb — Data Cleaning  
  * Cleans the injury dataset by filtering to the study window (October 2015 onward) and removing non-workload injury events: return-to-play activations, rest decisions, illness/COVID protocols, and contact injuries (contusions, fractures, concussions). Retains muscle, ligament, back, foot, and overuse injuries. Merges cleaned game logs into a single labeled dataset saved to data/processed/.


* 03_feature_engineering.ipynb — Feature Engineering
  * Engineers 22 features from game-level data only (no biometric or GPS data). All rolling windows use closed="left" to prevent data leakage. 
  * Workload features: rolling minutes over 7, 14, and 28-day windows; 5-game rolling total; cumulative season minutes; Acute:Chronic Workload Ratio (ACWR) and a binary danger-zone flag (ACWR >= 1.5). 
  * Schedule features: rest days since last game, back-to-back flag, games played in the last 7 days. 
  * Player features: age at game date, prior injury count, and 10 body-region recent injury flags (hamstring, quad, calf, groin, ankle, Achilles, knee, back, hip, foot) using a 56-day lookback window joined via merge_asof. 
  * Saves the feature matrix to data/processed/game_logs_features.csv.

* 05_modeling.ipynb — ML Modeling
  * Trains and evaluates four classifiers — Logistic Regression, Decision Tree, Random Forest, and XGBoost using a chronological train/test split (train: 2015-16 to 2020-21). 
  * Handles class imbalance with SMOTE and weighting. 
  * Hyperparameter tuning via GridSearchCV with TimeSeriesSplit. Evaluates on ROC-AUC, Average Precision, F1, Precision, and Recall. 
  * Saves trained models to models/.
