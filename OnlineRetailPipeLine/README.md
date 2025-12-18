# Online Retail Data Lake Pipeline (PySpark & Medallion Architecture)

This project implements an end‑to‑end **large‑scale data lake pipeline** for the classic **Online Retail** dataset using **PySpark** and a **Medallion (Bronze → Silver → Gold)** architecture.

The pipeline cleans and enriches raw transaction data, builds business KPI tables, applies clustering, and generates visualizations for exploratory analysis.

---

1. Project Overview

- **Dataset**: Online Retail transactional data (`Online Retail.csv`, 541,909 rows)
- **Objective**:
  - Ingest raw CSV data into a Bronze layer
  - Clean and standardize into a high‑quality Silver layer
  - Aggregate into Gold KPI tables for analytics
  - Run clustering (K‑Means) on daily performance
  - Visualize key trends

- **Tech Stack**
  - Python, PySpark
  - Jupyter / Colab notebook
  - `pandas`, `matplotlib`, `seaborn`
  - `pyspark.sql`, `pyspark.ml`

---

2. Data & Environment

2.1 Environment Setup (Colab)

Main setup (inside the notebook):

```python  
!apt-get install -y openjdk-17-jdk  
!pip install pyspark  

import os  
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-17-openjdk-amd64"  

2.2 Raw Data (Bronze)
File: Online Retail.csv

Example columns:

InvoiceNo (string)
StockCode (string)
Description (string)
Quantity (long)
InvoiceDate (string)
UnitPrice (double)
CustomerID (double)
Country (string)
Bronze is created by loading the CSV into a Spark DataFrame and profiling:

Null counts per column
Distinct counts
Min / max / average for Quantity, UnitPrice
Distinct Country values

3. Silver Layer – Cleaned & Enriched Data
3.1 Standardization
Column names standardized to:

InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country

Type fixes:

python
CustomerID  -> integer
InvoiceDate -> timestamp (format: "M/d/yyyy H:mm")

3.2 Data Cleaning
Filters applied:

Remove cancelled invoices (InvoiceNo starting with "C")
Remove rows with:
CustomerID is null
Quantity <= 0 or null
UnitPrice <= 0 or null
InvoiceDate null
Row counts

Bronze: 541,909 rows
Silver: 530,104 rows
Removed: 11,805 rows
Reasons for removal

Cancelled invoices: 9,288
Non‑positive quantity: 10,624
Non‑positive price: 2,517
(Overlaps across rules)

3.3 Enrichment
Added time and business fields:

InvoiceYear, InvoiceMonthNum, InvoiceDay
InvoiceHour, InvoiceMinute
DayOfWeek (string), IsWeekend (boolean)
WeekOfYear, Quarter
InvoiceMonth (e.g. 2011-09, used for partitioning)
Revenue = Quantity * UnitPrice
Silver is written as Parquet and partitioned by InvoiceMonth:

python
SILVER_PATH = "/content/silver"
silver.write.mode("overwrite").partitionBy("InvoiceMonth").parquet(SILVER_PATH)

4. Gold Layer – KPI Tables
Gold tables are derived from Silver and stored under /content/gold/.

4.1 KPI 1 – Daily Revenue Trends (daily_revenue)
Aggregations by InvoiceDate_Date:

TotalRevenue
TotalTransactions
UniqueCustomers
AvgRevenuePerTransaction
TotalItemsSold
Key findings

Average daily revenue ≈ 34,973
Highest day revenue ≈ 200,920.60 (2011‑12‑09)
Lowest day revenue ≈ 3,457.11
Top revenue days cluster in Q4 2011, indicating strong seasonality
Some days with few transactions but very high revenue (large wholesale‑like orders)

4.2 KPI 2 – Hourly Sales Patterns (hourly_patterns)
Grouped by InvoiceHour:

TotalRevenue
TransactionCount
AvgTransactionValue
UniqueCustomers
TotalQuantity
PeakPeriod flag: "Peak" if 10–15h, else "Non‑Peak"
Key findings

Busiest hour: 10:00 with ≈ 1.45M revenue
Peak period (10:00–15:00) accounts for the majority of revenue
Very little activity before 7am or after 8pm

4.3 KPI 3 – Country Performance (country_performance)
Aggregated per Country:

TotalRevenue
TotalOrders
UniqueCustomers
AvgOrderValue
TotalItemsSold
UniqueProductsPurchased
RevenuePerCustomer
Key findings

United Kingdom dominates:
≈ 84–85% of total revenue
Revenue per customer ≈ 2,302
38 countries represented
Some very small markets (e.g., Saudi Arabia) contribute ~0.001% of revenue

4.4 KPI 4 – Monthly Revenue Trends (monthly_trends)
Grouped by InvoiceYear, InvoiceMonthNum, InvoiceMonth:

TotalRevenue
TotalTransactions
UniqueCustomers
AvgTransactionValue
TotalItemsSold
UniqueProducts
RevenuePerCustomer
Key findings

Strong Q4 seasonality (Sept–Nov 2011 all above ~950k)
Revenue growth from first (2010‑12) to last month (2011‑12) ≈ −22.5%
Peak month: Nov 2011 (~1.15M revenue)
4.5 KPI 5 – Customer Behavior (customer_metrics)
Grouped by CustomerID:

TotalRevenue

TotalOrders

ActiveDays (distinct purchase days)

AvgOrderValue

TotalItemsPurchased

FirstPurchase

LastPurchase

CustomerLifespanDays

CustomerSegment:

High-Value : TotalRevenue ≥ 10,000
Medium-Value: 2,500–10,000
Low-Value : < 2,500
Key findings

High‑value customers (~100, ~2–3% of all) generate ~40% of revenue
High‑value average ≈ 35k per customer
Medium‑value (~600) ≈ 4.3k each
Low‑value (~3.6k, 90%+ of base) ≈ < 725 each
Top 10 customers alone ≈ 1.46M revenue

5. Machine Learning – Daily Revenue Clustering
Built on the daily_revenue Gold table:

Features:

TotalRevenue
TotalTransactions
Pipeline:

VectorAssembler → KMeans(k=4)
Outputs:

Cluster assignments per day

Cluster centers; example:

Cluster 0: [56,754.6, 2,588.3]
Cluster 1: [31,432.8, 1,666.0]
Cluster 2: [16,437.1, 1,085.0]
Cluster 3: [113,596.1, 2,641.1]
WSSSE (training cost): ~2.32e10

Interpretation: clusters separate low, medium, high, and exceptional revenue days.

6. Visualizations
Using matplotlib and seaborn:

Daily revenue time series (Gold KPI 1)
Hourly revenue – Peak vs Non‑Peak (Gold KPI 2)
Distribution of daily revenue (histogram)
Top 10 countries by revenue (bar chart)
Cluster size bar chart (number of days per K‑Means cluster)
Scatter: TotalTransactions vs TotalRevenue colored by cluster
These plots highlight seasonality, daily volatility, and segment behavior.

7. How to Run the Notebook
Open in Google Colab (recommended)
Upload Online Retail.csv to the Colab environment (e.g., /content/Online Retail.csv)
Run cells in order:
Environment setup
Bronze loading & profiling
Silver cleaning & enrichment
Gold KPI generation
Machine learning & visualization
Parquet outputs are written under /content/silver, /content/gold, /content/ml (can be mapped to S3 in a cloud setup).

8. Repository Structure (Suggested)
text
.
├── OnlineRetail_Pipeline.ipynb
├── data/
│   └── Online Retail.csv      # (optional, or stored externally)
├── README.md
└── figs/                      # optional: saved charts
9. Author
Ling Wang
End‑to‑end pipeline design, data cleaning, KPI modeling, clustering, and visualization.
