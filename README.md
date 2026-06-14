# Azure E-Commerce Data Pipeline

End-to-end Azure Data Engineering pipeline using Medallion Architecture, Delta Lake, and Databricks on 1M+ records.

## Architecture

![Architecture](architecture.png)

**Bronze → Silver → Gold** (Medallion Architecture)

## Tech Stack

- **Ingestion**: Azure Data Factory, Azure Storage SDK
- **Storage**: Azure Data Lake Storage Gen2
- **Transformation**: Azure Databricks, Apache Spark, PySpark
- **Data Format**: Delta Lake
- **Analytics**: Databricks SQL, Unity Catalog
- **Security**: Azure Key Vault, Managed Identity, RBAC
- **Orchestration**: Azure Data Factory

## Pipeline Overview

### Bronze Layer
- Raw 1M+ row global e-commerce dataset ingested as-is into ADLS Gen2
- No transformations, preserves source data integrity

### Silver Layer
- Explicit schema casting (no inferSchema)
- Null handling and deduplication (removed 8,193 duplicates)
- Boolean flag conversions (Yes/No → true/false)
- Business columns added:
  - `profit_margin` = (profit / revenue) × 100
  - `high_value_order_flag` = orders > $500
  - `fraud_flag` = fraud_risk_score > 75
  - `order_week` = week of year
- Star schema modeling:
  - `fact_orders` — 991,930 rows, partitioned by year/month
  - `dim_customer` — partitioned by country
  - `dim_product` — partitioned by category

### Gold Layer
- Business-ready aggregated Delta tables:
  - `revenue_by_country_category` — 3,748 rows
  - `monthly_sales_trends` — 126 rows
  - `customer_segment_analysis` — 60 rows
  - `fraud_risk_summary` — 750 rows

## Data Quality Checks
- Zero duplicate orders verified
- Zero null values on key columns (order_id, customer_id, product_id)
- Zero invalid prices
- Fraud rate: ~25% (synthetic dataset characteristic)
- High value orders: ~28%

## Security
- Azure Key Vault for secret management
- Managed Identity for passwordless authentication
- RBAC role assignments (Storage Blob Data Contributor)
- Unity Catalog External Locations for governed storage access

## Sample Queries

```sql
-- Top revenue by country and category
SELECT country, category, SUM(total_revenue) as revenue
FROM gold_revenue
GROUP BY country, category
ORDER BY revenue DESC
LIMIT 10;

-- Monthly sales trend
SELECT order_year, order_month, SUM(total_revenue) as revenue
FROM gold_monthly_trends
GROUP BY order_year, order_month
ORDER BY order_year, order_month;
```

## Setup

1. Clone this repository
2. Set up Azure resources (ADLS Gen2, Databricks, Key Vault)
3. Store credentials in Azure Key Vault
4. Run notebooks in order:
   - `01_ingest_kaggle_to_bronze.py`
   - `02_silver_production.py`
   - `03_gold_aggregations.py`

## Dataset
[Global E-Commerce Dataset 1M Records](https://www.kaggle.com/datasets/akrambelha/global-e-commerce-dataset-1m-records-20242026)