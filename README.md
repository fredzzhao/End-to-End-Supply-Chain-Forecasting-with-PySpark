# Demand Forecasting & Inventory Optimization Engine (PySpark)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fredzzhao/End-to-End-Supply-Chain-Forecasting-with-PySpark/blob/main/Inventory_Optimization_Engine.ipynb)

> **Personal project** on the public **Kaggle Superstore Sales** dataset, built to practice an end-to-end forecasting pipeline in PySpark. It contains no employer data.

The pipeline turns 9,800 raw order lines (2015–2018) into **weekly sales forecasts for 17 product sub-categories**, checks them against simple baselines, and converts forecast error into **safety buffers at a 95% service level**.

> All values are **sales dollars**, not units: the dataset has no quantity column.

---

## Results

Test period: 2017–2018 (1,768 sub-category-weeks). Training: 2015–2016 (1,717).

| Model | RMSE ($/week) | MAE ($/week) | Random Forest RMSE improvement |
|---|---:|---:|---:|
| **Random Forest (100 trees)** | **1,310** | **641** | — |
| 4-week moving average | 1,424 | 719 | 8.0% |
| Naive (last week's sales) | 1,772 | 822 | 26.1% |

**Safety buffers scale with volatility** (Z × RMSE, Z = 1.65):

| Sub-category | Forecast RMSE | Safety buffer ($/week) |
|---|---:|---:|
| Copiers | 3,226 | 5,322 |
| Machines | 2,110 | 3,481 |
| Chairs | 1,693 | 2,794 |
| … | … | … |
| Envelopes | 175 | 289 |
| Fasteners | 166 | 274 |

---

## Pipeline

1. **Ingestion:** reads the CSV with RFC 4180 quoting (`escape='"'`), so product names with inch marks (`14 7/8"" x 11""`) parse correctly; all 9,800 rows load.
2. **Cleaning & quality check:** casts sales to numeric, parses day-first dates, and reports invalid values before anything is dropped (0 found).
3. **Weekly aggregation:** sums order lines into weekly sales per sub-category (Monday-start weeks) on a **complete calendar grid**: 209 weeks × 17 sub-categories, with 867 no-sales weeks filled as 0.
4. **Feature engineering:** Spark window functions partitioned by sub-category: 1-week lag, 4-week lag, average of the **previous** 4 weeks, and week of year. No feature uses the target week.
5. **Modeling:** Spark MLlib Random Forest Regressor on a **time-based split** (train before 2017-01-02, test after), fixed seed for reproducibility.
6. **Evaluation:** RMSE and MAE against a naive and a 4-week moving-average baseline.
7. **Inventory policy:** per-sub-category RMSE × 1.65 = safety buffer; forecast + buffer = weekly requirement, output as a replenishment plan table.

## What changed in v2 (Oct 2026)

A review of v1 found three issues, all fixed:
- **CSV parsing:** Spark's default backslash escape misread 292 rows (3%) whose product names contain doubled quotes; v1 dropped them as "malformed". v2 reads them correctly.
- **Target leakage:** v1's rolling average included the week being predicted (`rowsBetween(-3, 0)`), which made its error look lower than it was. v2 uses `rowsBetween(-4, -1)`.
- **Missing weeks:** v1 skipped weeks with no sales, so a "1-week lag" could reach back several weeks. v2 zero-fills a complete weekly grid.

v2 also adds the two baselines and a fixed seed. The v2 error is higher than v1's, and it is the honest one.

## Limitations
- Sales dollars, not units (no quantity column).
- Safety buffers are sized from test-period error; a production setup would size them from a rolling validation window and add supplier lead time.
- One pooled model across sub-categories; no price, promotion or holiday features.

## Tech Stack
- **Language:** Python
- **Engine:** Apache Spark 4.0.1 (PySpark: Spark SQL, window functions, MLlib), local mode
- **Environment:** Google Colab

## How to Run
1. Open `Inventory_Optimization_Engine.ipynb` in Google Colab.
2. Upload `train.csv` (link below) to Google Drive and set `DATA_PATH` in section 2.
3. Run all cells. The first cell installs PySpark 4.0.1 and mounts Drive.

## Dataset
- **Source:** [Superstore Sales Dataset](https://www.kaggle.com/datasets/rohitsahoo/sales-forecasting) (Kaggle, Rohit Sahoo): 9,800 order lines with order date, ship mode, segment, region, category, sub-category and sales.

---
*Author: Fred Zhao*
