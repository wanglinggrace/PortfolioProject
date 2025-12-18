# Data Analytics Portfolio

This repository showcases three end‑to‑end data projects covering **e‑commerce**, **data engineering / pipelines**, and **real estate analytics**.  
Each project includes data cleaning, exploratory analysis, modeling, and business‑oriented insights.

## Projects Overview

| # | Project | Domain | Main Skills |
|---|---------|--------|------------|
| 1 | Etsy Jewelry Market Analysis | E‑commerce | Data cleaning, EDA, regression, Lasso |
| 2 | Online Retail Data Pipeline | Data Engineering | PySpark, Medallion architecture, KPIs, clustering |
| 3 | USA Real Estate Price Analysis | Real Estate | EDA, geospatial plots, Lasso, Decision Tree |

---

## 1. Etsy Jewelry Market Analysis

**Folder:** `EstyJewelryAnalysis/`  
**Goal:** Understand the Etsy jewelry market and what drives sales and revenue for top shops and listings.

**Key Points**

- Data from **eRank**: top Etsy jewelry shops and their listings.
- Cleaned and merged shop‑level and listing‑level Excel files.
- Created metrics such as:
  - shop **sales density** (sales per year)
  - listing‑level **average daily sales**
- EDA:
  - Sales concentration (Pareto: top ~62 shops ≈ 80% of sales)
  - Category analysis (earrings, necklaces, bracelets, rings)
  - Price distributions and price vs. sales
  - Correlations between views, favorites, sales, and revenue
- Modeling:
  - **Lasso regression**:
    - Features: total views, favorites, listing age, price
    - Target: estimated sales
    - \(R^2\) on test ≈ 0.93
    - Views are by far the most important feature.

**Business Insights**

- Market is **winner‑take‑most**, with a few dominant shops.
- **Personalized jewelry** (names, initials, birthstones) dominates top sellers.
- **Traffic & engagement** (views, favorites) matter much more than price.
- Recommendations for:
  - new sellers (start with personalized items, optimize for traffic)
  - existing sellers (focus on listings with strong engagement).

_For full details, see the README inside `EstyJewelryAnalysis/`._

---

## 2. Online Retail Data Pipeline (PySpark & Medallion Architecture)

**Folder:** `OnelineRetailPipeLine/`  
**Goal:** Build a **Bronze → Silver → Gold** data pipeline for the classic Online Retail dataset using **PySpark**, then derive business KPIs and clustering.

**Key Points**

- Environment: Google Colab / Jupyter with **PySpark**.
- **Bronze**:
  - Raw `Online Retail.csv` (≈ 541k rows).
  - Loaded and profiled (nulls, distincts, basic stats).
- **Silver**:
  - Cleaned and standardized:
    - removed cancelled invoices and invalid quantities/prices
    - converted dates and customer IDs to proper types
  - Added features:
    - date parts, day of week, weekend flag, week of year, quarter
    - `Revenue = Quantity * UnitPrice`
  - Written as partitioned Parquet.
- **Gold** KPI tables:
  - `daily_revenue`
  - `hourly_patterns`
  - `country_performance`
  - `monthly_trends`
  - `customer_metrics`
- **Clustering**:
  - K‑Means on daily revenue and transaction counts to segment days into low/medium/high/spike revenue clusters.
- **Visualizations**:
  - daily revenue series
  - hourly peak vs non‑peak patterns
  - top countries by revenue
  - cluster scatterplots.

**Business Insights**

- Strong **seasonality**, especially in Q4.
- Sales heavily driven by a small number of **very large orders**.
- UK dominates revenue; a small group of high‑value customers drives a big share of sales.

_For full details, see the README inside `OnlineRetailPipeLine/`._

---

## 3. USA Real Estate Price Analysis

**Folder:** `USAEstateAnalysis/`  
**Goal:** Analyze a large U.S. real estate listings dataset and build models to predict property prices from features and state.

**Key Points**

- Data: Realtor.com dataset (~2.2M rows down to ~1.57M after cleaning).
- Cleaning:
  - filtered unrealistic prices and house sizes
  - removed “ready to build” listings
  - dropped unused text columns
  - handled missing lot size with median imputation.
- EDA:
  - price distributions (right‑skewed with long high‑price tail)
  - median price by state, beds, baths, house size, lot size
  - correlations between price, size, beds/baths, lot size.
- Geospatial:
  - **Folium choropleth** maps for:
    - median price by state
    - number of listings by state.
- Modeling:
  - **Lasso regression** on log‑price:
    - Features: beds, baths, lot size, house size, status, state dummies
    - Test \(R^2 \approx 0.55\)
    - Key drivers: house_size, baths, and high‑price states (CA, WA, MA, FL, NY).
  - **Decision Tree Regressor**:
    - Tuned `max_depth`, best around 16
    - Test \(R^2 \approx 0.58\)
    - Top features: baths, house_size, and high‑price states.

**Business Insights**

- **Interior size** and **bathrooms** are stronger price drivers than lot size.
- State‑level location premiums are very significant.
- Models capture meaningful variation but show there is still large unexplained variance (missing finer‑grained location & property details).

_For full details, see the README inside `USAEstateAnalysis/`._

---

## Repository Structure (Suggested)

```text  
.  
├── EstyJewelryAnalysis/  
│   ├── data/  
│   ├── notebooks/  
│   └── README.md  
├── OnlineRetailPipeLine/  
│   ├── data/           
│   ├── notebooks/  
│   └── README.md  
├── USAEstateAnalysis/  
│   ├── data/  
│   ├── notebooks/  
│   └── README.md  
└── README.md           # this top-level file  
