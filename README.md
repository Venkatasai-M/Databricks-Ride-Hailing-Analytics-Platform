# Ride-Hailing Trip Analytics Platform (Medallion Architecture)

## Executive Summary
An end-to-end modern Data Engineering pipeline constructed on Databricks utilizing the Medallion Architecture (Bronze, Silver, Gold), Delta Lake, and Lakeflow Spark Declarative Pipelines. The platform ingests messy raw cab trip exports across eight Indian cities, cleanses anomalies, enforces data quality constraints, achieves 100% mathematical audit reconciliation, and prepares curated analytical views powering an Executive AI/BI Operations Dashboard[cite: 1, 2, 6].

---

## Architecture & Data Flow

```text
[Source CSV] (Unity Catalog Volume: ride_trips_raw.csv)
       │
       ▼
[Bronze Layer] (bronze_ride_trips - Streaming Table)
       │
       ├──► [Silver Layer: Clean] (silver_trips_clean - Materialized View)
       │         │  - Deduplication (SELECT DISTINCT)
       │         │  - Normalization (TRIM, INITCAP)
       │         │  - Imputation (COALESCE for fare & unknown city/payment)
       │         │  - Quality Constraints (EXPECT distance > 0, rate >= 0)
       │         │
       │         └──► [Gold Layer: Curated Analytics]
       │                   ├── gold_monthly_trips (BR-01)
       │                   ├── gold_city_summary (BR-02)
       │                   ├── gold_vehicle_summary (BR-03)
       │                   └── gold_payment_summary (BR-04)
       │
       ├──► [Silver Layer: Quarantined] (silver_trips_rejected - Materialized View)
       │         │  - Bad distance (<= 0) or negative rate (< 0) with root cause reason
       │
       └──► [Gold Layer: Audit & DQ] (gold_data_quality - BR-05 & BR-06)
                 │  - 100% Row Reconciliation Check: 1,214 = 1,168 + 22 + 24
                 ▼
     [Databricks AI/BI / Power BI 3-Page Dashboard]
