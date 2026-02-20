# 📊 Project Status Report — TMDB "The Next Blockbuster"

**Report Date:** February 19, 2026  
**Repository:** [github.com/lauramanzanos23/TMDB-Movies-INSY-674](https://github.com/lauramanzanos23/TMDB-Movies-INSY-674)  
**Course:** INSY 674

---

## 1. Project Overview

This project builds an **end-to-end machine learning pipeline** to predict movie success **before release** using data from The Movie Database (TMDB) API. The pipeline covers data extraction, cleaning, feature engineering, exploratory data analysis, and a Streamlit-based UI prototype.

**Business Goal:** Help studios and streaming platforms estimate whether a movie will be a "hit" and forecast expected popularity, based only on pre-release attributes (cast, director, genre, runtime, etc.).

---

## 2. Repository Structure

```
TMDB-Movies-INSY-674/
├── README.md                           # Project overview & documentation
├── PROJECT_STATUS_REPORT.md            # This report
├── app/
│   └── app_mockup.py                   # Streamlit UI prototype
├── data/
│   ├── movies_2010_2025.csv            # Raw extracted dataset
│   ├── data_cleaned_engineered.csv     # Cleaned + feature-engineered dataset
│   ├── data_advanced_features.csv      # (branch) Advanced features dataset
│   ├── data_quality_validated.csv      # (branch) Quality-validated dataset
│   └── data_semi_supervised_predictions.csv # (branch) Model predictions
├── EDA/
│   └── EDA.ipynb                       # Exploratory Data Analysis notebook
└── notebooks/
    ├── DataExtraction.ipynb            # TMDB API data extraction pipeline
    ├── FeatureEngineering.ipynb        # Base feature engineering & cleaning
    ├── AdvancedFeatureEngineering.ipynb # (branch) Advanced feature engineering
    ├── DataQualityValidation.ipynb     # (branch) Data quality & outlier treatment
    └── SemiSupervisedModel.ipynb       # (branch) Semi-supervised ML model
```

---

## 3. Branch Strategy

| Branch | Purpose | Status |
|--------|---------|--------|
| `main` | Stable project code — extraction, EDA, feature engineering, app mockup | **Active** |
| `Maria` | Contributor branch | Exists on remote |
| `feature/advanced-feature-engineering` | Adds advanced feature engineering notebook | **Pushed to GitHub** |
| `feature/data-quality-validation` | Adds data quality validation & outlier treatment notebook | **Pushed to GitHub** |
| `feature/semi-supervised-model` | Semi-supervised ML model for revenue tier prediction | **Pushed to GitHub** |

---

## 4. Work Completed (Pre-Existing on `main`)

### 4.1 Data Extraction (`notebooks/DataExtraction.ipynb`)
- Authenticated API calls to TMDB (`/discover/movie`, `/movie/{id}`, `/person/{id}`)
- Extracted movies from **2010–2025**, sorted by popularity (~30 pages per year)
- Enriched each movie with **credits** (top 5 actors + director) and **keywords**
- Person-level caching to respect API rate limits
- **Output:** `data/movies_2010_2025.csv`

### 4.2 Feature Engineering (`notebooks/FeatureEngineering.ipynb`)
- Removed duplicates by `movie_id`
- Parsed dates → `release_year`, `release_month`, `release_quarter`, `release_dayofweek`, `is_weekend_release`
- Normalized numeric columns; replaced zero budget/revenue with NaN
- Created missing-value indicator flags (`runtime_missing`, `budget_missing`, etc.)
- Text features from overview (`overview_len`, `overview_word_count`)
- Parsed and one-hot encoded top 15 genres and top 25 keywords
- Cast aggregate features (`actor_pop_mean/max/min/std`, `cast_size`)
- Gender representation counts (`gender_female_count`, `has_female_director`)
- Financial features (`log_budget`, `log_revenue`, `roi`, `success_revenue`, `success_roi_1_5`)
- Median imputation for missing numeric values (excluding target columns)
- **Output:** `data/data_cleaned_engineered.csv`

### 4.3 Exploratory Data Analysis (`EDA/EDA.ipynb`)
- Dataset shape and info inspection
- Missing value analysis (null counts + zero-value counts for actor popularity)
- Helper functions for parsing list columns and counting empties
- Numeric summary statistics
- Visualizations: distributions, genre breakdowns, correlation analysis, scatter plots
- Popularity and revenue trend analysis

### 4.4 Streamlit App Mockup (`app/app_mockup.py`)
- Mock predictor function simulating ML output
- User inputs: genre, actor tier, runtime, release month
- Outputs: hit probability, expected popularity score, explanation
- Styled UI with gradient cards and responsive layout

---

## 5. New Work Completed (Feature Branches)

### 5.1 Advanced Feature Engineering (`feature/advanced-feature-engineering`)

**Notebook:** `notebooks/AdvancedFeatureEngineering.ipynb`

| # | Feature Category | Features Created | Description |
|---|-----------------|-----------------|-------------|
| 1 | Seasonality flags | `is_summer_release`, `is_holiday_release`, `is_valentines_release`, `is_halloween_release`, `is_dump_month` | Binary flags for key release windows |
| 2 | Competition density | `monthly_competition`, `weekly_competition` | Count of movies released in the same month/week |
| 3 | Director track record | `director_hist_revenue`, `director_hist_vote_average`, `director_hist_popularity`, `director_film_count`, `is_debut_director` | Rolling historical averages (leak-free, excludes current movie) |
| 4 | Franchise detection | `is_franchise_keyword`, `is_sequel_title`, `is_franchise` | Sequel/franchise flags via keyword matching + title regex |
| 5 | Budget tiers | `budget_tier`, `budget_tier_*` (one-hot) | Categorized into micro / low / medium / high / blockbuster |
| 6 | Cast diversity | `cast_gender_diversity`, `female_ratio` | Shannon entropy index + female representation ratio |
| 7 | Text features | `avg_word_length`, `long_word_ratio`, `has_question`, `exclamation_count`, `sentence_count` | NLP-lite signals from movie overview |
| 8 | Genre interactions | `genre_action_x_comedy`, `genre_action_x_scifi`, `genre_horror_x_comedy`, `genre_drama_x_romance`, `genre_action_x_adventure`, `genre_animation_x_family`, `genre_crime_x_thriller` | Interaction features for common genre pairings |
| 9 | Engagement ratio | `popularity_per_vote`, `log_vote_count`, `is_high_hype_low_engagement` | Hype vs. actual viewer engagement signals |

**Output:** `data/data_advanced_features.csv`

### 5.2 Data Quality Validation (`feature/data-quality-validation`)

**Notebook:** `notebooks/DataQualityValidation.ipynb`

| # | Check Category | What It Does |
|---|---------------|--------------|
| 1 | Schema validation | Verifies column data types match expectations; auto-converts dates |
| 2 | Missing data analysis | Visualizes missing percentages (bar chart) + missing patterns (heatmap) |
| 3 | Missing data correlation | Correlation matrix of missingness to detect MAR/MNAR patterns |
| 4 | Outlier detection (IQR) | Identifies outliers using 1.5× IQR bounds for key numeric columns |
| 5 | Outlier detection (z-score) | Flags values beyond 3 standard deviations |
| 6 | Outlier treatment | Conservative winsorization (capping at 3× IQR) with `_capped` columns |
| 7 | Duplicate detection | Exact duplicates (by `movie_id`) + near-duplicates (same title + year) |
| 8 | Domain rules | Validates: runtime 1–600 min, votes 0–10, non-negative budget/revenue, year range, ROI sanity |
| 9 | Correlation analysis | Heatmap of key features + identifies pairs with |r| > 0.85 |
| 10 | Feature drop suggestions | Recommends which correlated feature to drop based on target correlation |

**Output:** `data/data_quality_validated.csv`

### 5.3 Semi-Supervised Model (`feature/semi-supervised-model`)

**Notebook:** `notebooks/SemiSupervisedModel.ipynb`

**Motivation:** Revenue is 72% missing — a perfect scenario for semi-supervised learning, which leverages both labeled and unlabeled data.

**Target:** Revenue Tier classification (Low / Medium / High / Blockbuster) based on quartiles of known revenue.

| # | Model | Type | Accuracy | Weighted F1 |
|---|-------|------|----------|-------------|
| 1 | Gradient Boosting | Supervised (baseline) | 61.4% | 61.6% |
| 2 | **Self-Training Classifier** | **Semi-Supervised** | **64.8%** | **64.7%** |
| 3 | Label Propagation | Semi-Supervised | 51.7% | 51.9% |
| 4 | Label Spreading | Semi-Supervised | 52.3% | 52.7% |

**Key Results:**
- Self-Training outperformed the supervised baseline by **+3.4% accuracy**, demonstrating the value of leveraging unlabeled data
- The model iteratively pseudo-labeled 6,628 unlabeled samples across 12 iterations
- Top predictive features: `vote_count`, `log_budget`, `budget`, `vote_average`, `popularity`
- Generated revenue tier predictions for **6,686 previously unlabeled movies** with 99.2% average confidence
- Includes confusion matrices, feature importance charts, and confidence distribution visualizations

**Output:** `data/data_semi_supervised_predictions.csv`

---

## 6. Commit History Summary

| Commit | Branch | Description |
|--------|--------|-------------|
| `17b00e7` | main | Initial commit |
| `06f35b4` | main | Create folders and add data |
| `8713837` | main | Project structure |
| `9b0a427` | main | Data extraction notebook |
| `850a690` | main | app.py file created |
| `595d2db` | Maria | App mockup |
| `0da1084` | main | Data upload |
| `6dc8c84` | main | Data upload movies 2020–2025 |
| `9c72a18` | main | Data upload movies 2010–2025 |
| `bd7c553` | main | Data extraction notebook |
| `10a82db` | main | EDA |
| `d6189be` | main | Feature engineering draft |
| `736b2dd` | feature/advanced-feature-engineering | Advanced feature engineering notebook |
| `cdceb98` | feature/data-quality-validation | Data quality validation notebook |
| `b96905f` | feature/semi-supervised-model | Semi-supervised model: Self-Training, Label Propagation, Label Spreading |

---

## 7. Next Steps

- [ ] Merge feature branches into `main` via Pull Requests
- [x] Build semi-supervised classification model (revenue tier prediction)
- [ ] Build regression model (predict continuous popularity score)
- [ ] Evaluate additional models (ROC-AUC, MAE, RMSE, R²)
- [ ] Connect trained models to the Streamlit app (replace mock predictor)
- [ ] Final documentation and presentation

---

## 8. Technologies Used

| Category | Tools |
|----------|-------|
| Language | Python 3.x |
| Data | Pandas, NumPy |
| ML | Scikit-learn (GradientBoosting, SelfTraining, LabelPropagation, LabelSpreading) |
| Visualization | Matplotlib, Seaborn |
| API | TMDB API (requests) |
| App | Streamlit |
| Version Control | Git, GitHub |
| Environment | Jupyter Notebooks, VS Code |
