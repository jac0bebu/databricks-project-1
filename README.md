# 🚲 Bike Lakehouse — Data Engineering Project

An end-to-end data engineering pipeline that ingests raw CRM and ERP data through a **Bronze → Silver → Gold** Medallion Architecture using Databricks, PySpark, Delta Lake, and Unity Catalog.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Project Structure](#project-structure)
- [Data Layers](#data-layers)
- [Getting Started](#getting-started)

---

## 📌 Project Overview

This project builds a scalable **Lakehouse** platform that:

- Ingests raw data from **CRM** (Customer Relationship Management) and **ERP** (Enterprise Resource Planning) source systems
- Applies **data cleaning, standardization, and transformation** across layers
- Delivers **analytics-ready data** for business intelligence and reporting
- Enforces **data governance** through Unity Catalog

---

## 🏛️ Architecture

```
Source Systems
┌─────────────┐     ┌─────────────┐
│     CRM     │     │     ERP     │
│  customers  │     │  products   │
│  sales      │     │  finance    │
└──────┬──────┘     └──────┬──────┘
       │                   │
       ▼                   ▼
┌─────────────────────────────────┐
│         BRONZE LAYER            │
│   Raw data ingestion (as-is)    │
│   workspace.bronze.*            │
└────────────────┬────────────────┘
                 │  Clean & Standardize
                 ▼
┌─────────────────────────────────┐
│         SILVER LAYER            │
│   Cleaned, typed, conformed     │
│   workspace.silver.*            │
└────────────────┬────────────────┘
                 │  Aggregate & Model
                 ▼
┌─────────────────────────────────┐
│          GOLD LAYER             │
│   Business-ready, Star Schema   │
│   workspace.gold.*              │
└─────────────────────────────────┘
                 │
                 ▼
        BI Tools / Analytics
```

---

## 🛠️ Technologies Used

- Databricks  
- Apache Spark  
- PySpark  
- Spark SQL  
- Delta Lake  
- Unity Catalog 

---

## 📁 Project Structure

```
bike-lakehouse/
│
├── datasets/
│   ├── source_crm/
│   │   ├── cust_info.csv           # Customer profiles
│   │   ├── prd_info.csv            # Product catalog
│   │   └── sales_details.csv       # Sales transactions
│   │
│   └── source_erp/
│       ├── CUST_AZ12.csv           # Customer account data
│       ├── LOC_A101.csv            # Location and country data
│       └── PX_CAT_G1V2.csv         # Product categories
│
├── scripts/
│   ├── Bronze/
│   │   └── Bronze                  # Bronze ingestion notebook
│   │
│   ├── Silver/
│   │   ├── crm/
│   │   │   ├── silver_crm_cus_info         # Clean CRM customers
│   │   │   ├── silver_crm_prd_info         # Clean CRM products
│   │   │   └── silver_crm_sale_details     # Clean CRM sales
│   │   ├── erp/
│   │   │   ├── silver_erp_cus_az12         # Clean ERP customers
│   │   │   ├── silver_erp_loc_a101         # Clean ERP locations
│   │   │   └── silver_erp_px_cat_g1v2      # Clean ERP categories
│   │   └── silver_orchestration            # Run all silver scripts
│   │
│   └── Gold/
│       ├── gold_dim_customers              # Customer dimension table
│       ├── gold_dim_products               # Product dimension table
│       ├── gold_fact_sales                 # Sales fact table
│       └── gold_orchestration              # Run all gold scripts
│
└── README.md
```

---



## 🥉🥈🥇 Data Layers

This project follows the **Medallion Architecture**:

```
Bronze (Raw)        Silver (Cleaned)              Gold (Business)
────────────        ────────────────              ───────────────
crm_cust_info   →   crm_cust_info_cleaned     →   dim_customer
crm_prd_info    →   crm_prd_info_cleaned      →   dim_product
crm_sales       →   crm_sales_cleaned         →   fact_sales
erp_cust_az12   →   erp_cust_az12_cleaned     →   dim_location
erp_loc_a101    →   erp_loc_a101_cleaned
erp_px_cat      →   erp_px_cat_cleaned
```

| Layer | Purpose | Schema |
|---|---|---|
| 🥉 **Bronze** | Raw data injestion, Schema inference and storage as Delta tables | `workspace.bronze` |
| 🥈 **Silver** |Data cleaning and standardization, Type casting and validation | `workspace.silver` |
| 🥇 **Gold** | Business Transformation, Star Schema | `workspace.gold` |

---

### Bronze — Raw Ingestion
- Exact copy of source data, no transformation
- Stored as **Delta tables** in `workspace.bronze.*`
- Schema: `catalog → bronze → table`

### Silver — Cleaned & Conformed
Transformations applied:
- ✅ Trim whitespace from all string columns
- ✅ Standardize values (`"S"` → `"Single"`, `"F"` → `"Female"`)
- ✅ Split composite columns (`prd_key` → `prd_cat` + `prd_key`)
- ✅ Handle NULL values with `coalesce()`
- ✅ Validate and cast date columns (`yyyyMMdd` → `DateType`)
- ✅ Derive missing prices from `sales / quantity`
- ✅ Expand abbreviations (`"US"` → `"United States"`)

### Gold — Business Ready
- Star Schema modeling (Fact + Dimension tables)
- Pre-aggregated metrics for BI tools
- Stored in `workspace.gold.*`

| Table | Type | Description |
|---|---|---|
| `gold_dim_customers` | Dimension | Clean customer profiles |
| `gold_dim_products` | Dimension | Clean product catalog |
| `gold_fact_sales` | Fact | Sales transactions with keys |

---

## ⚙️ Pipeline — Databricks Jobs

The project uses **Databricks Jobs** to orchestrate the full pipeline:

```
Job: loading_bike_data_lakehouse

bronze_layer  →  silver_layer  →  gold_layer
     │                │                │
  Bronze/           Silver/          Gold/
  Bronze          silver_          gold_
               orchestration   orchestration
```

Each layer runs sequentially — Bronze must complete before Silver starts, Silver before Gold.

| Task | Script | 
|---|---|
| `bronze_layer` | `scripts/Bronze/Bronze` |
| `silver_layer` | `scripts/Silver/silver_orchestration` |
| `gold_layer` | `scripts/Gold/gold_orchestration` | 
---

## 🚀 Getting Started

### Prerequisites
- Databricks workspace with Unity Catalog enabled
- GitHub account linked to Databricks
- Cluster with Databricks Runtime 13.0+

### Setup

1. **Clone the repository**
```bash
git clone https://github.com/jac0bebu/databricks-project-1.git
```

2. **Link repo to Databricks**
```
Databricks Workspace
→ Workspace → Add → Git Folder
→ Paste repo URL
→ Create Git Folder
```

3. **Run manually layer by layer**
```
scripts/Bronze/Bronze                    ← Bronze first
scripts/Silver/silver_orchestration      ← Silver second
scripts/Gold/gold_orchestration          ← Gold last
```

4. **Or run the full pipeline via Databricks Job**
```
Databricks → Jobs → loading_bike_data_lakehouse → Run Now
```

*Built with ❤️ using Databricks Lakehouse Platform*
