# Retail Medallion Lakehouse

A Bronze → Silver → Gold data pipeline on the UCI Online Retail II dataset, built with Databricks and delivered as an interactive Power BI dashboard.

## Architecture

<img width="682" height="936" alt="image" src="https://github.com/user-attachments/assets/3c7f03ab-6408-4d03-a741-b0310d34b774" />


Raw transactional data is ingested, cleaned, and modeled into a star schema across three Delta Lake layers, then served to Power BI via DirectQuery.

## Dataset

[UCI Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) — ~1,067,371 raw transaction rows from a UK-based online retailer, covering multiple countries and two years of sales.

## Pipeline

### Bronze — raw ingest
- `01_bronze_ingest`: reads the source Excel file, standardizes schema/types, writes `bronze_online_retail`

### Silver — cleaned + validated
- `02_silver_clean`: runs 6 data quality checks (cancelled invoices, duplicate rows, negative quantity, null customer ID, null description, zero/negative price), logs results to `dq_results`
- Adds flag columns rather than deleting rows, preserving full lineage
- Output: `silver_online_retail` (1,067,371 rows)

### Gold — star schema
- `03_gold_star_schema`: builds `dim_date` (739 rows), `dim_customer` (5,942 rows), `dim_product` (4,932 rows), and `fact_sales` (797,736 rows, returns kept as negative-value rows)
- Zero orphan joins validated across all fact-to-dimension relationships

## Power BI dashboard

Connected via SQL Warehouse + Personal Access Token, using **DirectQuery** — the model stays live against the Gold tables rather than importing a snapshot.

**Pages:**
1. **Sales Overview** — KPI cards, net sales trend by month, country and date-range filters
2. **Top Products** — top 10 products by net sales, with supporting detail table
3. **Customer Segments** — RFM (Recency, Frequency, Monetary) segmentation built entirely in DAX: quartile scoring, segment labels (Champions / Loyal / Potential / At Risk / Lost), distribution chart, top customers table, and average value comparisons by segment
4. **DQ Health** — data quality KPIs and failure rates by check, with a trend view designed to track drift across pipeline runs over time

## Tech stack

- **Databricks Free Edition** — PySpark, Delta Lake, Unity Catalog, Git-linked notebooks
- **Power BI Desktop** — DirectQuery, DAX
- **GitHub** — version control for all notebooks

## Notes

Building RFM segmentation entirely in DAX under DirectQuery (rather than in the notebook) surfaced a few non-obvious constraints worth documenting:
- DirectQuery blocks iterator functions (`RANKX`, `SUMX`, `MAXX`) in calculated columns, so all scoring logic lives in measures instead
- `RANKX`'s tie-handling parameter matters: `DENSE` inflates scores on low-cardinality fields like day-counts; `Skip` gives correct quartile splits
- Measures can't be used directly as chart Legend or axis grouping fields — worked around using a small static lookup table combined with `SELECTEDVALUE` + `FILTER`

## Repository structure

```text
retail-medallion-lakehouse/
├── 01_bronze_ingest.ipynb
├── 02_silver_clean.ipynb
├── 03_gold_star_schema.ipynb
├── README.md
├── Retail-Medallion-Lakehouse.pbit
└── Screenshots/
    ├── sales-overview.png
    ├── top-products.png
    ├── customer-segments.png
    └── dq-health.png
```
