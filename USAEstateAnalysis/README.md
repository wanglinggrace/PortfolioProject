# USA Real Estate Analysis with Machine Learning

This repository contains an end‑to‑end analysis of residential real estate listings across the United States, using a large Realtor.com dataset and several machine learning models.

We explore how property characteristics and location relate to housing prices and build predictive models to estimate property values.

---

## 1. Project Overview

- **Goal**
  - Explore patterns in the U.S. housing market.
  - Understand which features drive house prices.
  - Build models to predict prices from property characteristics and state.

- **Dataset**
  - Source: Realtor.com (file: `realtor-data.zip.csv`)
  - Size: ~2.2M rows, 12 columns.

- **Main Tools**
  - Python: `pandas`, `numpy`
  - Visualization: `matplotlib`, `seaborn`, `folium`, `geopandas`
  - Machine Learning: `scikit-learn` (Lasso, Decision Tree)

---

## 2. Data Description

Original columns:

- `brokered_by` – broker or agency
- `status` – listing status (`for_sale`, `sold`, `ready to build`, etc.)
- `price` – listing price (USD)
- `bed` – number of bedrooms
- `bath` – number of bathrooms
- `acre_lot` – lot size in acres
- `street`, `city`
- `state` – U.S. state / territory
- `zip_code`
- `house_size` – interior size in square feet
- `prev_sold_date` – previous sale date

Initial dataset shape:

- \(2226382\) rows  
- 12 columns

---

## 3. Data Cleaning & Preparation

Main cleaning steps:

1. **Filter unrealistic values**
   - `price` between 1,000 and 5,000,000.
   - `house_size` between 300 and 15,000 square feet.

2. **Filter by status**
   - Keep records where `status != 'ready to build'`.

3. **Drop unused columns**
   - Removed: `prev_sold_date`, `brokered_by`, `street`, `city`, `zip_code`.

4. **Remove duplicates**
   - Used `df.drop_duplicates()`.

5. **Handle missing values**
   - `acre_lot`: filled missing values with the median.

6. **Check remaining nulls and unique values**
   - `status`: `for_sale`, `sold`
   - `state`: 50+ states and territories
   - `bed`, `bath`, `acre_lot`, `house_size`, `price` have wide ranges.

Final dataset shape after cleaning:

- ~1.57M rows  
- 7 columns: `status`, `price`, `bed`, `bath`, `acre_lot`, `state`, `house_size`.

---

## 4. Exploratory Data Analysis (EDA)

### 4.1 Price Distribution

- Plotted histogram of `price` (20 bins).
- Prices are **right‑skewed** with many lower‑ to mid‑priced properties and a long tail of expensive homes.

### 4.2 Median Price by State

- Computed median `price` for each `state`.
- Visualized using bar plot and a **Folium choropleth** map.
- Some states (e.g., **California, Washington, Massachusetts, New York, Florida**) have significantly higher median prices.

### 4.3 Median Price by Bedrooms and Bathrooms

- Grouped by `bed` and `bath`, computed median `price`.
- Visualized with bar charts.

Findings:

- More bedrooms and bathrooms are associated with higher median prices.
- Relationship is generally increasing but not perfectly linear, especially at extreme values.

### 4.4 Median Price by House Size and Lot Size

- Binned `house_size` into 1,000 sq ft ranges.
- Binned `acre_lot` into 0.1 acre ranges.
- Plotted median price for each bin.

Findings:

- **House size** has a strong positive relationship with price.
- **Lot size** shows a weaker but still positive relationship.

### 4.5 House Size and Listings by State

- Median `house_size` by `state`.
- Count of listings by `state`.

Findings:

- Some states have larger average houses (e.g., certain western and southern states).
- Listing counts are highest in large population states.

### 4.6 Correlation Analysis

- Computed numeric correlation matrix for:
  - `price`, `bed`, `bath`, `acre_lot`, `house_size`.
- Visualized with a heatmap.

Key correlations:

- `house_size` vs `bath`: strong positive.
- `house_size` vs `price`: strong positive.
- `bath` vs `price`: strong positive.
- `bed` vs `price`: moderate positive.
- `acre_lot` has near‑zero correlation with other variables and with price.

---

## 5. Geospatial Visualization

Used **Folium** and a U.S. states GeoJSON:

1. **State Abbreviations**
   - Mapped full state names to 2‑letter abbreviations and stored in `state_abbrev`.

2. **Median House Price by State**
   - Grouped by `state_abbrev`, computed median `price`.
   - Created a choropleth map:
     - `fill_color = 'RdYlGn'`, `fill_opacity = 0.6`.

3. **Number of Listings per State**
   - Counted listings per `state_abbrev`.
   - Created a second choropleth map to show listing density.

These maps highlight regional differences in prices and listing volume.

---

## 6. Machine Learning Models

We built two main regression models to predict log prices.

### 6.1 Feature Engineering

- One‑hot encoded categorical variables:
  - `status`
  - `state`
- Target variable:
  - \(y = \log(1 + \text{price})\) to reduce skew.
- Feature matrix `X`:
  - `bed`, `bath`, `acre_lot`, `house_size`,
  - dummy variables for `status` and `state`.
- Dropped `price` and `state_abbrev` from `X`.

Train–test split:

- 75% training, 25% test (`train_test_split`, `random_state=42`).

Standardization:

- Used `StandardScaler` on `X`.

---

### 6.2 Lasso Regression

- Model:
  - `LassoCV(cv=5, max_iter=10000)` to find optimal alpha.
  - Best alpha ≈ 0.000431.
- Trained final model:
  - `Lasso(alpha=best_alpha, max_iter=100000)`.

Performance:

- Training \(R^2 \approx 0.545\)
- Test \(R^2 \approx 0.546\)

Feature importance (by absolute coefficient):

- Most important features:
  - `state_California`
  - `house_size`
  - `bath`
  - `state_Washington`
  - `state_Massachusetts`
  - `state_Florida`
  - `state_NewYork`
- `acre_lot` and many smaller states have minimal influence.

Interpretation:

- Both **size** (especially `house_size` and `bath`) and **state‑level location** strongly impact price.
- Lasso captures meaningful linear structure but still leaves substantial unexplained variance.

---

### 6.3 Decision Tree Regressor

- Tried max depths from 1 to 19.
- Plotted training and test \(R^2\) vs `max_depth` to monitor overfitting.
- Best depth:
  - `max_depth = 16` (highest test score).

Final model:

- `DecisionTreeRegressor(random_state=0, max_depth=16)`.

Performance:

- Training \(R^2 \approx 0.621\)
- Test \(R^2 \approx 0.576\)

Feature importances:

- Highest importance:
  - `bath` (≈ 0.44)
  - `house_size` (≈ 0.17)
  - `state_California` (≈ 0.16)
  - `acre_lot`
  - `state_Washington`
  - `bed`
  - `state_Massachusetts`
  - `state_Florida`
  - `state_NewYork`
- Many states have very small but non‑zero importance.

Interpretation:

- Tree captures **non‑linear relationships** and interactions.
- Slightly higher test \(R^2\) than Lasso, with some overfitting.

---

## 7. Key Findings

- **Property size** is crucial:
  - `bath` and `house_size` are the strongest predictors of price in both models.
- **Location premium**:
  - States like California, Washington, Massachusetts, Florida, and New York have higher median prices and large positive model coefficients.
- **Lot size**:
  - `acre_lot` has weaker correlation and smaller importance than interior size and bathrooms.
- **Model performance**:
  - Lasso: Test \(R^2 \approx 0.55\)
  - Decision Tree: Test \(R^2 \approx 0.58\)
  - There is still considerable unexplained variation, suggesting missing features (e.g., city, zip code, age of property, neighborhood quality).

---

## 8. Repository Structure (Example)

You can adapt this to match your files:

```text  
.  
├── data/  
│   └── realtor-data.zip.csv           # raw dataset (not committed if too large)  
├── notebooks/  
│   └── GroupProjectUSAEstate.ipynb    # main analysis notebook  
├── figs/  
│   ├── price_distribution.png  
│   ├── median_price_by_state.png  
│   ├── correlations_heatmap.png  
│   ├── lasso_feature_importance.png  
│   └── tree_feature_importance.png  
└── README.md  
