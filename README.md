# Data Warehouse — Medallion Architecture

Data Warehouse built on SQL Server using the **Medallion architecture** (Bronze → Silver → Gold) to consolidate CRM and ERP data into an analytical model.

---

## Layers

| Layer | Type | Responsibility |
|-------|------|----------------|
| Bronze | Physical tables | Raw ingestion from CSV files, no transformation |
| Silver | Physical tables | Cleansing, deduplication, standardization |
| Gold | Views | Star Schema ready for analytics and reporting |

---

## Repository Structure

```
├── init_database.sql           # Database and schema setup
├── ddl_bronze.sql              # Bronze table definitions
├── ddl_silver.sql              # Silver table definitions
├── ddl_gold.sql                # Gold views (Star Schema)
├── proc_load_bronze.sql        # ETL: source → Bronze
├── proc_load_silver.sql        # ETL: Bronze → Silver
├── quality_silver_checks.sql   # Silver quality checks
├── quality_gold_checks.sql     # Gold quality checks
└── gold_report_customer.sql    # Customer analytics view
```

---

## Data Sources

| System | File | Table | Content |
|--------|------|-------|---------|
| CRM | `cust_info.csv` | `bronze.crm_cust_info` | Customer records |
| CRM | `prd_info.csv` | `bronze.crm_prd_info` | Product catalog |
| CRM | `sales_details.csv` | `bronze.crm_sales_details` | Sales transactions |
| ERP | `CUST_AZ12.csv` | `bronze.erp_cust_az12` | Birthdate and gender |
| ERP | `LOC_A101.csv` | `bronze.erp_loc_a101` | Customer country |
| ERP | `PX_CAT_G1V2.csv` | `bronze.erp_px_cat_g1v2` | Product categories |

---

## Data Model

See the draw.io diagrams in `/docs` for the full architecture and Star Schema.

| Table | Type | Description |
|-------|------|-------------|
| `gold.dim_customers` | Dimension | Consolidated customer data from CRM and ERP |
| `gold.dim_products` | Dimension | Active products with categories |
| `gold.fact_sales` | Fact | Sales transactions linked to dimensions |

---

## Silver Transformations

| Table | Key Transformations |
|-------|-------------------|
| `crm_cust_info` | Deduplication by `cst_id`; gender and marital status normalization; TRIM on text fields |
| `crm_prd_info` | Category ID extraction from product key; product line normalization; SCD Type 2 end date via `LEAD()` |
| `crm_sales_details` | Integer-to-date conversion (`YYYYMMDD`); price and sales amount recalculation when inconsistent |
| `erp_cust_az12` | `NAS` prefix removal; birthdate range validation; gender normalization |
| `erp_loc_a101` | Hyphen removal from customer ID; country name standardization |

---

## How to Run

> **Warning:** `init_database.sql` drops and recreates the `DataWarehouse` database.

```sql
-- 1. Setup
init_database.sql
ddl_bronze.sql
ddl_silver.sql
ddl_gold.sql

-- 2. Load
EXEC bronze.load_bronze
EXEC silver.load_silver

-- 3. Validate
quality_silver_checks.sql
quality_gold_checks.sql
```

CSV files must be accessible by the SQL Server service at `/datasets/source_crm/` and `/datasets/source_erp/`.

---

## Requirements

- SQL Server 2016+ or Azure SQL Database
- `BULK INSERT` permission for the load user
- Read access to the CSV source directories
