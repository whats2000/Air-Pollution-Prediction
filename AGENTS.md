# PM2.5 Time-Series Forecasting: Taiwan Air Quality Hourly Dataset (Oct–Dec)

## Objective
Using **October and November** as the training set and **December** as the test set, predict:
- The **next 1-hour PM2.5** value, and  
- The **next 6-hour PM2.5** value

based on the **previous 6 hours** of pollutant data.  
Models: **Linear Regression** and **XGBoost**  
Metric: **Mean Absolute Error (MAE)**  

You must produce **8 MAE results** in total:  
(2 feature types × 2 forecast horizons × 2 models)

---

## Overview
**Workflow:**
Data Preprocessing → Time-Window Construction → Model Training → Evaluation (MAE) → Final Submission

---

## Data Structure and Submission Format

### Dataset Information
The dataset (provided `.xls` file) contains **hourly pollutant readings** across 18 features:

| Feature    | Description                 |
|------------|-----------------------------|
| AMB_TEMP   | Ambient Temperature         |
| CH4        | Methane                     |
| CO         | Carbon Monoxide             |
| NMHC       | Non-Methane Hydrocarbons    |
| NO         | Nitric Oxide                |
| NO2        | Nitrogen Dioxide            |
| NOx        | Nitrogen Oxides             |
| O3         | Ozone                       |
| PM10       | Particulate Matter (10 μm)  |
| PM2.5      | Particulate Matter (2.5 μm) |
| RAINFALL   | Precipitation               |
| RH         | Relative Humidity           |
| SO2        | Sulfur Dioxide              |
| THC        | Total Hydrocarbons          |
| WD_HR      | Hourly Wind Direction       |
| WIND_DIREC | Wind Direction              |
| WIND_SPEED | Wind Speed                  |
| WS_HR      | Hourly Wind Speed           |

Each day contains **24 hourly readings (0–23)**.

### Data Validity and Missing Values
Symbols indicating invalid data:
- `#` → Instrument invalid  
- `*` → Program invalid  
- `x` → Manually invalid  
- `A` → Instrument alarm (invalid)  
- *blank* → Missing value  
- `NR` → “No Rain” → should be replaced with **0**

### Dataset Partition
- **Training set:** October + November (61 days × 24 hours = 1464 hours)  
- **Test set:** December (31 days × 24 hours = 744 hours)

### Time-Window Layout
Each feature forms a continuous hourly sequence:
- Training matrix: **(18 × 1464)**  
- Test matrix: **(18 × 744)**

### Prediction Targets and Sample Counts
| Target             | Offset (hours)              | Train samples    | Test samples   |
|--------------------|-----------------------------|------------------|----------------|
| Next 1 hour (t+1)  | 6-hour window → predict t+1 | 1464 - 6 = 1458  | 744 - 6 = 738  |
| Next 6 hours (t+6) | 6-hour window → predict t+6 | 1464 - 11 = 1453 | 744 - 11 = 733 |

### Feature Variants
1. **PM2.5-only:**  
   - Each sample has 6 features (past 6 hours of PM2.5)
2. **All 18 attributes:**  
   - Each sample has 18 × 6 = 108 features (past 6 hours of all pollutants)

### Submission Format
- Columns: `No`, `PM2.5`
- Rows: 738 (corresponding to December’s t+1 forecast)
- Save your final predictions as **`submission.csv`**

---

# Step 1. Data Preprocessing

### Goal
Clean and standardize hourly pollutant data, handle missing or invalid values, and split by month.

### Instructions
1. **Load dataset** from `_2021.xls`.  
2. **Filter** only October, November, and December records.
3. **Clean invalid symbols:**
   - Replace `#`, `*`, `x`, `A`, and blanks with `NaN`
   - Replace `NR` in `RAINFALL` with `0`
4. **Convert** all columns to numeric (`float`), coercing non-numeric to `NaN`.
5. **Handle missing values:**
   - Replace NaN with the **mean of the previous and next 1 hour**
   - If the previous hour is also missing, look further back until a valid value is found.
   - If no previous valid value exists, use the next valid one.
6. **Reshape data:**
   - Combine hourly rows for each month into a 2D DataFrame:  
     - Rows = 18 features  
     - Columns = total hourly timestamps  
7. **Split data:**
   - Training matrix: October + November (18×1464)
   - Test matrix: December (18×744)
8. **Validation check:**
   - Confirm all 18 features have 1464/744 valid values after imputation.
   - Verify time continuity (no missing hourly timestamps).

### Expected Outputs
✅ Cleaned matrices:  
`train_matrix_18x1464` and `test_matrix_18x744`

---

# Step 2. Time-Series Window Construction

### Goal
Create rolling-window datasets for supervised learning.

### Instructions
1. **Define window length:** 6 hours  
2. **Targets:**
   - **t+1 forecast:** predict PM2.5 at hour `t+6`
   - **t+6 forecast:** predict PM2.5 at hour `t+11`
3. **Feature construction:**
   - **PM2.5-only:** `[PM2.5[t-6], ..., PM2.5[t-1]]` → 6 features
   - **All-18:** `[all_features[t-6:t-1]]` → 108 features
4. **Sample counts:**
   - Train: 1458 (t+1), 1453 (t+6)
   - Test: 738 (t+1), 733 (t+6)
5. **Data separation rule:**
   - Never mix training (Oct–Nov) and test (Dec)
   - If you normalize features, **fit only on training**, then apply to test.

### Expected Outputs
✅ 4 training datasets + 4 test datasets:
- PM2.5-only × (t+1, t+6)
- All-18 × (t+1, t+6)

---

# Step 3. Model Training

### Goal
Train both **Linear Regression** and **XGBoost** models on all four dataset variants.

### Instructions

#### Linear Regression
- Apply **feature standardization** (`StandardScaler`) before fitting.
- Train on each of the four combinations:
  1. PM2.5-only, t+1  
  2. PM2.5-only, t+6  
  3. All-18, t+1  
  4. All-18, t+6  

#### XGBoost Regressor
- Use `XGBRegressor` with reasonable defaults:
  - `n_estimators=500`, `max_depth=6`, `learning_rate=0.05`, `subsample=0.8`
- Use **early stopping** on a validation split from the training set.
- Fix `random_state` for reproducibility.

#### Model naming
| Model   | Feature | Target | Name        |
|---------|---------|--------|-------------|
| Linear  | PM2.5   | t+1    | LR_pm25_h1  |
| Linear  | PM2.5   | t+6    | LR_pm25_h6  |
| Linear  | All-18  | t+1    | LR_all_h1   |
| Linear  | All-18  | t+6    | LR_all_h6   |
| XGBoost | PM2.5   | t+1    | XGB_pm25_h1 |
| XGBoost | PM2.5   | t+6    | XGB_pm25_h6 |
| XGBoost | All-18  | t+1    | XGB_all_h1  |
| XGBoost | All-18  | t+6    | XGB_all_h6  |

### Expected Outputs
✅ 8 trained models  
✅ Predictions on both training and test sets

---

# Step 4. Evaluation (MAE)

### Goal
Compute and compare Mean Absolute Error (MAE) for all 8 models.

### Instructions
1. **Compute MAE** for test sets (Dec data).
2. **Build a summary table**:

| Feature    | Target | Model   | MAE |
|------------|--------|---------|-----|
| PM2.5-only | t+1    | Linear  | ... |
| PM2.5-only | t+1    | XGBoost | ... |
| PM2.5-only | t+6    | Linear  | ... |
| PM2.5-only | t+6    | XGBoost | ... |
| All-18     | t+1    | Linear  | ... |
| All-18     | t+1    | XGBoost | ... |
| All-18     | t+6    | Linear  | ... |
| All-18     | t+6    | XGBoost | ... |

3. **Interpret the results:**
   - Compare PM2.5-only vs All-18 → effect of more features  
   - Compare Linear vs XGBoost → effect of model complexity  
   - Compare t+1 vs t+6 → effect of forecast horizon length  

4. **Optional visualizations:**
   - Residual plots (error vs hour)
   - Feature importance (for XGBoost)
   - MAE bar chart comparison

### Expected Outputs
✅ MAE comparison table  
✅ Insights about model and feature performance

---

# Step 5. Final Inference and Submission

### Goal
Generate the competition submission file (`submission.csv`).

### Instructions
1. **Select the model** for official submission:
   - Task requirement: **PM2.5-only, next 1-hour (t+1)** forecast
2. **Retrain** the best-performing model using all training data (Oct–Nov).
3. **Predict** on the December test windows (738 samples).
4. **Prepare submission file:**
   - `No`: 0–737
   - `PM2.5`: predicted values
5. **Save file:**
   - Filename: `submission.csv`
   - Columns: `No`, `PM2.5`
   - Rows: 738

### Expected Output
✅ `submission.csv` — with correct structure for evaluation

---

# Step 6. Visualization and Reporting

### Recommended Plots
1. **Missing-value summary** before/after imputation (bar chart)
2. **PM2.5 time series** for Oct–Dec (line plot)
3. **Feature importance** (XGBoost)
4. **Residual vs Hour of Day**
5. **MAE comparison chart**

### Final Deliverables
- `notebook.ipynb` (complete workflow)
- `mae_table` (8 MAE scores)
- `submission.csv` (if required)
- Supporting plots and short discussion summary
