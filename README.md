
# Azure Retail CDC Data Pipeline

An end-to-end data engineering project simulating a production-style **Change Data Capture (CDC)** pipeline for retail order data, built using **Azure Databricks (PySpark, Delta Lake)** and **Azure Data Factory**, orchestrated with **Databricks Workflows** and monitored via **Azure Monitor**.

## 🏗 Architecture

Source Data → Bronze (raw) → Silver (validated + CDC merged) → Reconciliation ↓ ↓ ↓ Azure Data Factory triggers Databricks Workflow (Serverless Compute)


**Medallion Architecture**: Bronze (raw) → Silver (cleansed, deduplicated, CDC-merged) layers built on **Azure Data Lake Storage Gen2**, using **Delta Lake** for ACID transactions and time travel.

## 🎯 Key Features

- **CDC Pipeline**: Implements incremental upserts using Delta Lake's `MERGE INTO`, with Window-function-based deduplication to handle late-arriving updates (keeps latest record per business key).
- **Data Validation**: Automated schema validation and data quality checks (null detection, duplicate detection, range validation) on raw ingested data.
- **Performance Optimization**: Benchmarked and optimized Spark join performance using **partitioning** and **broadcast joins**, achieving a **measured 33% runtime reduction** (8.31s → 5.57s on a 326K-row synthetic dataset).
- **ETL Testing & Reconciliation**: Automated reconciliation checks comparing row counts, aggregate sums, referential integrity, and row-level hash comparisons between Bronze and Silver layers.
- **Orchestration**: Azure Data Factory triggers a Databricks Workflow (Job) containing sequential tasks: Validate → CDC Merge → Reconciliation — all running on **Databricks Serverless compute**.
- **Monitoring**: Azure Monitor alert rules configured to notify via email on pipeline failure.
- **CI/CD Ready**: ADF pipeline connected to Git version control (this repository) for change tracking.

## 🛠 Tech Stack

| Category | Technology |
|---|---|
| Compute | Azure Databricks (Serverless), PySpark |
| Storage | Azure Data Lake Storage Gen2 (ADLS Gen2), Delta Lake |
| Orchestration | Azure Data Factory, Databricks Workflows |
| Monitoring | Azure Monitor |
| Version Control / CI/CD | Git, Azure DevOps |

## 📓 Notebooks

| Notebook | Purpose |
|---|---|
| `01_test_connection.py` | Verifies Databricks ↔ ADLS Gen2 connectivity via Unity Catalog External Locations |
| `02_generate_load_bronze.py` | Generates synthetic retail order data, loads into Bronze layer |
| `03_validate_bronze.py` | Data quality validation report (nulls, duplicates, schema, range checks) |
| `04_bronze_to_silver_cdc.py` | Deduplication (Window functions) + Delta Lake CDC MERGE logic (Bronze → Silver) |
| `05_simulate_new_batch.py` | Simulates incoming incremental data (new + updated orders) to test CDC |
| `06_optimize_benchmark.py` | Benchmarks join performance: baseline vs. broadcast join vs. partitioning |
| `07_reconciliation.py` | ETL reconciliation checks between Bronze and Silver layers |

## 📊 Performance Results

| Optimization Stage | Runtime | Improvement |
|---|---|---|
| Baseline (no optimization) | 8.31s | — |
| + Broadcast Join | 6.48s | 21.97% |
| + Partitioning | 5.57s | **33.00%** |

*Benchmarked on a synthetically scaled dataset of ~326,400 rows joined against a 6-row lookup table.*

## 🔍 Data Quality & Reconciliation Results

- ✅ Schema validation: 8/8 columns matched expected structure
- ✅ Row count reconciliation: Silver row count matches expected distinct order_id count
- ✅ Sum reconciliation: Bronze and Silver aggregate totals matched exactly
- ✅ Referential integrity: 0 orphan records
- ✅ Row-level hash comparison: 0 mismatched records

## 📌 Notes

This project was built on Azure's free tier ($200 credit) using **Databricks Serverless compute** end-to-end. Several architecture decisions (e.g., using Databricks Job activity instead of ADF's classic Notebook activity, materializing intermediate data instead of `.persist()`/`.cache()`) were driven by Serverless-specific compatibility constraints discovered during development — documented as learning points in the notebook comments.

## 👤 Author

**Nitesh Choudhary**
[wwww.linkedin.com/in/nitesh810](#) | [Email](mailto:nitesh.ping@gmail.com)
