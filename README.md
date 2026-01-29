# End-to-End Supply Chain Demand Forecasting & Inventory Optimization Engine

## Project Overview
This project implements a distributed machine learning pipeline using **PySpark** to predict weekly sales demand and optimize inventory levels for a retail superstore. 

This project bridges the gap between Forecasting and Inventory Planning. It calculates dynamic **Safety Stock** levels based on model uncertainty to achieve a target **Service Level (95%)**.

## Key Business Features
* **Scalable Data Processing:** Utilizes PySpark to handle transactional data, simulating an environment capable of processing millions of records.
* **Time-Series Feature Engineering:** Implements advanced window functions to generate Lag features (Autocorrelation) and Rolling Statistics (Trend Analysis).
* **Dynamic Inventory Optimization:** Automatically generates purchase recommendations by calculating Cycle Stock and Safety Stock for each product category.

## Tech Stack
* **Language:** Python
* **Core Engine:** Apache Spark (PySpark)
* **Machine Learning:** Spark MLlib (Random Forest Regressor)
* **Environment:** Google Colab

## Methodology
The pipeline consists of five key stages:

1.  **Data Ingestion & Cleaning:** * Automated schema inference and type casting.
    * Robust handling of malformed records (ANSI SQL strict mode management).
2.  **Feature Engineering:** * Aggregated transactional data into `Weekly` granularity.
    * Created `Lag_1_Week`, `Lag_4_Weeks`, and `Rolling_Avg_4_Weeks` using Spark Window functions.
3.  **Modeling:** * Trained a **Random Forest Regressor** to capture non-linear relationships.
    * Employed **Time-Based Splitting** (Train < 2017, Test >= 2017) to prevent data leakage.
4.  **Evaluation:** * Evaluated using RMSE (Root Mean Square Error).
5.  **Inventory Strategy (The "Senior" Touch):** * Calculated category-specific volatility (Standard Deviation of Forecast Error).
    * Derived **Safety Stock** = $Z \times RMSE$ (where $Z=1.65$ for 95% Service Level).

## Results Snapshot
The model outputs a final **Replenishment Plan** table:

| Sub-Category | Cycle Stock (Predicted) | Safety Stock (Buffer) | Total Inventory Needed |
| :--- | :--- | :--- | :--- |
| Copiers | 3766 | 8266 | 12032 |
| Phones | 2100 | 500 | 2600 |
| ... | ... | ... | ... |

*(Note: Safety stock varies by category volatility. High-risk categories like 'Copiers' are assigned higher buffers.)*

## How to Run
This notebook is optimized for **Google Colab**.

1.  Open `Inventory_Optimization_Engine.ipynb` in Google Colab.
2.  Upload the `train.csv` (Superstore Dataset) to your Colab runtime or mount Google Drive.
3.  Run all cells. The environment will automatically install OpenJDK and PySpark.

## Dataset
* **Source:** [Superstore Sales Dataset on Kaggle](https://www.kaggle.com/datasets/rohitsahoo/sales-forecasting)
* **Description:** Historical transaction data including Order Date, Sales, Category, and Region.

---
*Author: Fred Zhao*
