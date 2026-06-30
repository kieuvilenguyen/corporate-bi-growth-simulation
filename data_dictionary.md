# Data Dictionary — ITB Business Intelligence Project

This document defines the schema of all datasets used in the ITB BI pipeline: raw source tables ingested in Notebook 1 and the processed Star Schema tables produced in Notebook 3.

---

## 1. Raw Source Datasets

### 1.1. Transaction Records (`dim_order`)

**Primary Key:** `order_id` | **Source:** 5 partitioned CSV files (`dim_order_part_1.csv` … `dim_order_part_5.csv`)

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `order_id` | Int64 | No | Unique transaction identifier. |
| `customer_id` | Int64 | No | Foreign reference to customer record. |
| `order_date` | String | No | Transaction date (YYYY-MM-DD). |
| `order_status` | String | No | Fulfillment status: `successfully delivered`, `cancelled after delivery`, `cancel in delivery`, `returned order`, `out of stock`. |
| `product_profile` | String | No | Product line: `product_A`, `product_B`, `product_C`. |
| `goods_category` | String | No | Product sub-category code. |
| `category_product_code` | String | No | Product category system code. |
| `product_code` | String | No | Unique SKU identifier. |
| `store_code` | Int64 | No | Physical store code. Negative values (`-1`, `-2`) denote online channel placeholders. |
| `product_cost` | Float64 | No | Unit COGS (VND). |
| `product_price` | Float64 | No | Unit retail price / revenue (VND). |
| `sales_channel` | String | No | Distribution channel: `website`, `ecommerce`, `stores`, `premium`. |

---

### 1.2. Customer Master (`dim_customer`)

**Primary Key:** `customer_id`

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `customer_key` | Int64 | No | Legacy surrogate key from source system. |
| `customer_id` | Int64 | No | Unique customer business identifier. |
| `name_full` | String | No | Anonymized full name. |
| `gender` | String | No | `Male`, `Female`, `XNA`. |
| `dob` | String | No | Date of birth (YYYY-MM-DD). |
| `address` | String | Yes | Residential address. |
| `region` | String | No | Province / City of residence. |
| `district` | String | Yes | District of residence. |
| `education_level` | String | Yes | Highest academic qualification. |
| `age` | Int64 | No | Age (pre-calculated). |
| `job_title` | String | Yes | Current occupation. |

---

### 1.3. Store Master (`dim_stores`)

**Primary Key:** None (product-level duplicates per store exist in source)

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `store_codes` | Float64 | Yes | Store identifier (includes online placeholders). |
| `sales_channel` | String | No | Channel classification (legacy hybrid value `stores/premium` present). |
| `product_profile` | String | Yes | Product profile distributed at the store. |
| `name_district` | String | Yes | Store district. |
| `name_region` | String | Yes | Store province / city. |

---

### 1.4. Product Catalog (`dim_product`)

**Primary Key:** `product_code`

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `category_product_code` | String | No | Product category system code. |
| `product_profile` | String | No | Product line: `product_A`, `product_B`, `product_C`. |
| `goods_category` | String | No | Product sub-category description. |
| `product_code` | String | No | Unique SKU identifier. |

---

## 2. Processed Star Schema (Data Warehouse)

Output of Notebook 3. Optimized for BI tool ingestion (Power BI / Tableau).

### 2.1. Fact Table (`fact_order`)

**Primary Key:** `order_id` | **Grain:** One row per transaction

| Column | Type | Nullable | FK Reference | Description |
| :--- | :--- | :--- | :--- | :--- |
| `order_id` | Int64 | No | — | Unique transaction identifier. |
| `customer_key` | Int64 | No | `dim_customer.customer_key` | Customer dimension surrogate key. |
| `product_key` | Int64 | No | `dim_product.product_key` | Product dimension surrogate key. |
| `store_key` | Int64 | Yes | `dim_stores.store_key` | Store dimension surrogate key. NULL for online transactions. |
| `date_key` | Int64 | No | `dim_date.date_key` | Date dimension key (YYYYMMDD). |
| `order_status_key` | Int64 | No | `dim_order_status.order_status_key` | Order status dimension surrogate key. |
| `channel_key` | Int64 | No | `dim_channel.channel_key` | Sales channel dimension surrogate key. |
| `product_cost` | Float64 | No | — | Unit COGS (VND). |
| `product_price` | Float64 | No | — | Unit retail price / revenue (VND). |

---

### 2.2. Customer Dimension (`dim_customer`)

**Primary Key:** `customer_key` (surrogate)

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `customer_key` | Int64 | No | Sequential surrogate key. |
| `customer_id` | Int64 | No | Original business identifier. |
| `name_full` | String | No | Anonymized full name. |
| `gender` | String | No | `Male`, `Female`, `XNA`. |
| `address` | String | No | Standardized address; nulls imputed as `Unknown`. |
| `region` | String | No | Province / City (Title Case). |
| `dob` | DateTime64 | No | Date of birth (YYYY-MM-DD). |
| `age` | Int64 | No | Verified chronological age. |
| `education_level` | String | No | Standardized qualification; nulls imputed as `Unknown`. |
| `job_title` | String | No | Standardized occupation; nulls imputed as `Unknown`. |

---

### 2.3. Product Dimension (`dim_product`)

**Primary Key:** `product_key` (surrogate)

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `product_key` | Int64 | No | Sequential surrogate key. |
| `product_code` | String | No | Unique SKU identifier. |
| `product_profile` | String | No | Product line: `product_A`, `product_B`, `product_C`. |
| `goods_category` | String | No | Product sub-category description. |
| `category_product_code` | String | No | Product category system code. |

---

### 2.4. Store Dimension (`dim_stores`)

**Primary Key:** `store_key` (surrogate) | **Scope:** Physical stores only; online channel codes excluded.

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `store_key` | Int64 | No | Sequential surrogate key. |
| `store_code` | Int64 | No | Physical store location code. |
| `store_region` | String | Yes | Province / City (Title Case). |
| `store_district` | String | Yes | District (Title Case). |

---

### 2.5. Date Dimension (`dim_date`)

**Primary Key:** `date_key` (YYYYMMDD integer)

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `date_key` | Int64 | No | Date integer key (YYYYMMDD format, e.g., `20241231`). |
| `fully_date` | DateTime64 | No | Full date timestamp. |
| `year` | Int32 | No | Year. |
| `quarter` | Int32 | No | Quarter (1–4). |
| `month` | Int32 | No | Month (1–12). |
| `day` | Int32 | No | Day of month (1–31). |
| `weekday` | String | No | Day name (e.g., `Wednesday`). |

---

### 2.6. Order Status Dimension (`dim_order_status`)

**Primary Key:** `order_status_key`

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `order_status_key` | Int64 | No | Sequential surrogate key. |
| `status_description` | String | No | Fulfillment status label (e.g., `successfully delivered`). |

---

### 2.7. Sales Channel Dimension (`dim_channel`)

**Primary Key:** `channel_key`

| Column | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `channel_key` | Int64 | No | Sequential surrogate key. |
| `channel_name` | String | No | Channel name: `website`, `ecommerce`, `stores`, `premium`. |