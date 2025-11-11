# Air-Pollution-Prediction

## Objective
Using **October and November** as the training set and **December** as the test set, predict:
- The **next 1-hour PM2.5** value, and
- The **next 6-hour PM2.5** value

based on the **previous 6 hours** of pollutant data.
Models: **Linear Regression** and **XGBoost**
Metric: **Mean Absolute Error (MAE)**

You must produce **8 MAE results** in total:
(2 feature types × 2 forecast horizons × 2 models)

## Overview
**Workflow:**
Data Preprocessing → Time-Window Construction → Model Training → Evaluation (MAE) → Final Submission

## Dataset
The dataset contains **hourly pollutant readings** across 18 features from the Taiwan Air Quality Monitoring Network.

**Data Source:** [Taiwan Air Quality Hourly Dataset](https://airtw.epa.gov.tw/CHT/Query/His_Data.aspx)

**Competition:** [HW3 Time-Series Regression Competition on Kaggle](https://www.kaggle.com/competitions/hw3-time-series-regression/overview)

### Features
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

### Data Partition
- **Training set:** October + November (61 days × 24 hours = 1464 hours)
- **Test set:** December (31 days × 24 hours = 744 hours)

### Feature Variants
1. **PM2.5-only:** 6 features (past 6 hours of PM2.5)
2. **All 18 attributes:** 108 features (past 6 hours of all pollutants)

## Workflow Steps

### Step 1. Data Preprocessing
- Load dataset from `_2021.xls`
- Filter October, November, and December records
- Clean invalid symbols: `#`, `*`, `x`, `A`, blanks → `NaN`; `NR` in `RAINFALL` → `0`
- Convert to numeric, handle missing values by interpolation
- Reshape to matrices: Train (18×1464), Test (18×744)

### Step 2. Time-Series Window Construction
- Window length: 6 hours
- Targets: t+1 and t+6 forecasts
- Create 4 datasets: PM2.5-only × (t+1, t+6), All-18 × (t+1, t+6)

### Step 3. Model Training
- **Linear Regression:** With StandardScaler
- **XGBoost:** With early stopping, n_estimators=500, max_depth=6, etc.
- Train 8 models total

### Step 4. Evaluation (MAE)
- Compute MAE on test sets
- Compare PM2.5-only vs All-18, Linear vs XGBoost, t+1 vs t+6

### Step 5. Final Inference and Submission
- Select best model for PM2.5-only, t+1
- Predict on December test (738 samples)
- Generate `submission.csv` with columns: `No`, `PM2.5`

### Step 6. Visualization and Reporting
- Missing-value summary
- PM2.5 time series
- Feature importance (XGBoost)
- Residual plots
- MAE comparison chart

## Installation

### Prerequisites
- Python 3.10 or higher
- uv (for dependency management)

### Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/whats2000/Air-Pollution-Prediction.git
   cd Air-Pollution-Prediction
   ```

2. Install dependencies using uv:
   ```bash
   uv sync
   ```

3. Activate the virtual environment:
   ```bash
   uv run python --version
   ```

## Usage

### Running the Notebook
1. Launch Jupyter Notebook:
   ```bash
   uv run jupyter notebook
   ```

2. Open `notebook.ipynb` and run the cells sequentially.

### Key Steps in the Notebook
1. **Data Preprocessing**: Load, clean, and reshape data
2. **Window Construction**: Create training/test datasets
3. **Model Training**: Train Linear Regression and XGBoost models
4. **Evaluation**: Compute MAE and compare models
5. **Submission**: Generate predictions and save to `submission.csv`

### Direct Execution
```bash
uv run jupyter nbconvert --to notebook --execute notebook.ipynb
```

## Dependencies
- `ipykernel`: Jupyter kernel
- `matplotlib`: Plotting
- `notebook`: Jupyter Notebook
- `numpy`: Numerical computing
- `pandas`: Data manipulation
- `scikit-learn`: Machine learning
- `scipy`: Scientific computing
- `tqdm`: Progress bars
- `xgboost`: XGBoost library
- `xlrd`: Excel file reading

## Results
The project produces 8 MAE scores comparing different models and feature sets. The final submission is `submission.csv` for the Kaggle competition.

## Contributing
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License
This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments
- Taiwan Environmental Protection Administration for the air quality data
- Kaggle for hosting the competition
- Open-source community for the libraries used
