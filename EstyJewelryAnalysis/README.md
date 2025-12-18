# Project 1: Etsy Jewelry Market Analysis

This project analyzes the Etsy jewelry market using top‐seller and listing‐level data from **eRank**.  
It focuses on understanding what drives sales and revenue for top Etsy jewelry shops and provides
actionable insights for new or existing sellers on product and pricing strategy.

---

## 1. Project Overview

- **Goal**
  - Understand market‑level dynamics in the Etsy jewelry category.
  - Identify how engagement metrics (views, favorites) relate to sales and revenue.
  - Provide recommendations for new and existing Etsy jewelry shops.

- **Tools & Tech**
  - Python: `pandas`, `numpy`, `seaborn`, `matplotlib`, `scikit-learn`
  - Jupyter Notebook for analysis and visualization

- **Data Sources**
  - **Market‑level file**: `eRank-Top Sellers.xlsx`  
    Each row is a shop with total estimated sales, year established, country and category.
  - **Listing‑level files**: exports for the top 10 jewelry shops (CaitlynMinimalist, MignonandMignon, GLDNxLayeredAndLong, etc.), combined into one dataset of listings.

---

## 2. Data Preparation

- Merged 10 shop‑level listing files into a single dataset.
- Cleaned numeric fields stored as text with symbols:
  - `EST.SALES`, `EST.REVENUE`, `PRICE`, `TOTAL VIEWS`, `FAVORITES`, `LISTING AGE`
- For shop‑level data:
  - Extracted 4‑digit `YEAR` and computed `STORE_AGE = 2025 − YEAR`
  - Calculated **Sales Density** = `SALES / STORE_AGE`
- For listing‑level data:
  - Created `AVG_DAILY_SALES = EST.SALES / LISTING AGE`
  - Dropped unused columns like `TAGS`, `LAST UPDATED`, `#`

---

## 3. Market‑Level Analysis

- **Sales concentration (Pareto)**
  - Top **62 shops account for ~80%** of total market sales.
  - Indicates a **winner‑take‑most** structure with a small group of dominant sellers.

- **Store age vs total sales**
  - Older stores tend to have higher cumulative sales, but:
    - Some young shops achieve very high sales.
    - Many older shops remain mid‑range performers.
  - Age helps, but success is not guaranteed by age alone.

- **Sales density (growth efficiency)**
  - Computed sales per year since establishment.
  - Identified high‑efficiency shops like **CaitlynMinimalist**, **MignonandMignon**, **OuferJewelry**, **BABEINA**.
  - These are useful **benchmarks** for new sellers.

---

## 4. Product‑Level Analysis

Listings were grouped into four main categories based on title text:

- **Earrings**
- **Necklaces**
- **Bracelets**
- **Rings**

### 4.1 Top products

For each category, the project:

- Selected **top 25 listings by estimated revenue**.
- Selected **top 25 listings by daily views**.
- Saved the results to Excel files and visualized them using bar charts.

Examples of findings:

- Many top products are **personalized** items:
  - Name necklaces, handwriting bracelets, custom birthstone pieces.
- A “top 50 best‑seller” table highlights the highest revenue items across all categories.

### 4.2 Price distribution

- Most products are priced in the **low‑to‑mid range** (roughly \$20–\$80).
- Very high‑priced items exist but are rare.

### 4.3 Price vs sales

- Simple linear regression of `EST.SALES` on `PRICE`:
  - Slope is slightly negative.
  - \(R^2 \approx 0.003\) → **almost no linear relationship**.
- Interpretation: success occurs at many price points; price alone does not explain sales.

### 4.4 Correlation analysis

- Strong positive correlations:
  - `EST.SALES` and `EST.REVENUE`
  - `TOTAL VIEWS`, `FAVORITES`, `DAILY VIEWS` with sales and revenue
- `PRICE` has only **weak** correlation with sales.
- Insight: **traffic and engagement (views, favorites) matter much more than price.**

### 4.5 Listing age and daily activity

- Some older listings still receive high daily views (long‑term best‑sellers).
- Many new listings start with low views and need time/marketing to gain traction.

### 4.6 Category performance

Summary by category (example values from analysis):

- **Necklaces**: highest total revenue and most listings.
- **Rings** and **earrings**: also large contributors.
- **Bracelets**: fewer listings but relatively high average prices.

---

## 5. Machine Learning Model

A simple **Lasso regression** model was built to explain `EST.SALES` using:

- Features: `TOTAL VIEWS`, `FAVORITES`, `LISTING AGE`, `PRICE`
- Target: `EST.SALES`

Results:

- Best \(\alpha \approx 614{,}041\)
- Training \(R^2 \approx 0.94\), Test \(R^2 \approx 0.93\)
- Feature importance (absolute coefficients):
  - `TOTAL VIEWS` is by far the most important driver.
  - `FAVORITES`, `LISTING AGE`, and `PRICE` contribute very little in the regularized model.

---

## 6. Key Insights & Recommendations

### Market‑level

- The Etsy jewelry market is **highly concentrated**: a small group of shops generates most sales.
- New entrants should:
  - Benchmark against **high sales‑density shops** rather than average performers.
  - Focus on efficient growth, not just longevity.

### Product‑level

- **Personalization wins**:
  - Name, initial, handwriting, and birthstone jewelry dominate top sellers.
- **Engagement over price**:
  - Increasing **views** and **favorites** is more important than small price changes.
  - Invest in product photography, SEO, and marketing.

### Category strategy

- Necklaces are the largest revenue pool.
- Rings and earrings are also strong categories.
- Bracelets can offer higher price points with fewer competitors.

### For new sellers

- Start with **personalized necklaces/rings/earrings**.
- Track your own **sales density** (sales per year) and compare with top shops.
- Prioritize traffic and engagement: improve titles, tags, photos, and social promotion.

### For existing shops

- Identify listings with good engagement but moderate revenue:
  - Test small price adjustments.
  - Improve descriptions/photos.
  - Consider bundles or upsells.
- Double down on product types that appear frequently among top sellers.

---

## 7. Repository Structure

.  
├── data/  
│   ├── eRank-Top Sellers.xlsx  
│   ├── CaitlynMinimalist-Shop listings.xlsx  
│   ├── ... (other top-seller listing files)  
├── notebooks/  
│   └── EtsyJewelryMarketAnalysis.ipynb  
├── outputs/  
│   ├── Top10sellerListings.xlsx  
│   ├── Top10sellerEarrings.xlsx  
│   ├── Top10sellerNecklaces.xlsx  
│   ├── Top10sellerBracelets.xlsx  
│   ├── Top10sellerRings.xlsx  
│   └── Top50BestSellerItems.xlsx  
└── README.md  
