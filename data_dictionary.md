# Data Dictionary - ITB Business Intelligence Project

This document provides a detailed description of the database schemas utilized in the ITB BI project, covering both the raw ingested datasets and the final processed Star Schema Data Warehouse.

---

## 1. Raw Ingested Datasets

These are the transactional and master datasets imported directly from source systems into the ETL pipeline.

### 1.1. Raw Transactions (`dim_order`)
* **Primary Key:** `order_id`
* **File Source:** Partitioned into 5 parts (`dim_order_part_1.csv` to `dim_order_part_5.csv`)

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `customer_id` | Int64 | No | Unique identifier of the purchasing customer. |
| `order_id` | Int64 | No | Unique identifier of the transaction. |
| `order_date` | Object (String) | No | Date of transaction (YYYY-MM-DD). |
| `order_status` | Object (String) | No | Operational status (`successfully delivered`, `cancelled after delivery`, `cancel in delivery`, `returned order`, `out of stock`). |
| `product_profile` | Object (String) | No | Product line classification (`product_A`, `product_B`, `product_C`). |
| `goods_category` | Object (String) | No | Code representing the sub-category of goods. |
| `category_product_code` | Object (String) | No | Product category system code. |
| `product_code` | Object (String) | No | Unique product stock keeping unit (SKU) identifier. |
| `store_code` | Int64 | No | Code representing physical store location (contains negative values like `-1`, `-2` as online placeholders). |
| `product_cost` | Float64 | No | Production cost (COGS) of the product unit in VND. |
| `product_price` | Float64 | No | Retail selling price of the product unit in VND. |
| `sales_channel` | Object (String) | No | Channel of sale (`website`, `ecommerce`, `stores`, `premium`). |

### 1.2. Raw Customer Master (`dim_customer`)
* **Primary Key:** `customer_id`

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `customer_key` | Int64 | No | Existing technical surrogate key from legacy system. |
| `customer_id` | Int64 | No | Unique business identifier of the customer. |
| `name_full` | Object (String) | No | Full name of the customer (anonymized/masked). |
| `gender` | Object (String) | No | Gender of the customer (`Male`, `Female`, `XNA`). |
| `dob` | Object (String) | No | Customer's date of birth (YYYY-MM-DD). |
| `address` | Object (String) | Yes | Full residential address. |
| `region` | Object (String) | No | Standardized Province/City of residence. |
| `district` | Object (String) | Yes | District of residence. |
| `education_level` | Object (String) | Yes | Customer's academic qualification level. |
| `age` | Int64 | No | Customer's age (pre-calculated). |
| `job_title` | Object (String) | Yes | Customer's current occupation. |

### 1.3. Raw Store Master (`dim_stores`)
* **Primary Key:** None (Contains product-level duplicates per store)

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `store_codes` | Float64 | Yes | Store business identifier (includes online placeholders). |
| `sales_channel` | Object (String) | No | Channel classification (contains legacy hybrid value `stores/premium`). |
| `product_profile` | Object (String) | Yes | Product profile registered to be distributed at the store. |
| `name_district` | Object (String) | Yes | District where the physical store is located. |
| `name_region` | Object (String) | Yes | Province/City where the physical store is located. |

### 1.4. Raw Product Catalog (`dim_product`)
* **Primary Key:** `product_code`

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `category_product_code` | Object (String) | No | Unique product category system code. |
| `product_profile` | Object (String) | No | Core product line (`product_A`, `product_B`, `product_C`). |
| `goods_category` | Object (String) | No | Product sub-category group description. |
| `product_code` | Object (String) | No | Unique SKU identifier. |

---

## 2. Processed Star Schema Data Warehouse

These tables represent the standardized structures exported by the ETL modeling pipeline (`fact_order` and clean dimensions) designed for optimal BI tool ingestion.

### 2.1. Central Fact Table (`fact_order`)
* **Primary Key:** `order_id`

| Column Name | Data Type | Nullable | Foreign Key Mapping | Description |
| :--- | :--- | :--- | :--- | :--- |
| `order_id` | Int64 | No | None | Unique transactional order identifier. |
| `customer_key` | Int64 | No | `dim_customer.customer_key` | Map to Customer dimension. |
| `product_key` | Int64 | No | `dim_product.product_key` | Map to Product dimension. |
| `store_key` | Int64 | Yes | `dim_stores.store_key` | Map to Store dimension (Null for online channels). |
| `date_key` | Int64 | No | `dim_date.date_key` | Map to Date dimension (YYYYMMDD). |
| `order_status_key` | Int64 | No | `dim_order_status.order_status_key` | Map to Order Status dimension. |
| `channel_key` | Int64 | No | `dim_channel.channel_key` | Map to Sales Channel dimension. |
| `product_cost` | Float64 | No | None | Production cost (COGS) of the unit. |
| `product_price` | Float64 | No | None | Selling price (revenue) of the unit. |

### 2.2. Customer Dimension (`dim_customer`)
* **Primary Key:** `customer_key` (Surrogate Key)

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `customer_key` | Int64 | No | Sequential surrogate key generated during modeling. |
| `customer_id` | Int64 | No | Original unique customer business ID. |
| `name_full` | Object (String) | No | Anonymized full name. |
| `gender` | Object (String) | No | Standardized gender (`Male`, `Female`, `XNA`). |
| `address` | Object (String) | No | Standardized address (NaN converted to `'Unknown'`). |
| `region` | Object (String) | No | Province/City formatted with Title Case. |
| `dob` | DateTime64 | No | Parsed date of birth (YYYY-MM-DD). |
| `age` | Int64 | No | Verified chronological age. |
| `education_level` | Object (String) | No | Standardized and unified academic qualification (NaN converted to `'Unknown'`). |
| `job_title` | Object (String) | No | Standardized occupation (NaN converted to `'Unknown'`). |

### 2.3. Product Dimension (`dim_product`)
* **Primary Key:** `product_key` (Surrogate Key)

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `product_key` | Int64 | No | Sequential surrogate key generated during modeling. |
| `product_code` | Object (String) | No | Unique product SKU code. |
| `product_profile` | Object (String) | No | Core product line (`product_A`, `product_B`, `product_C`). |
| `goods_category` | Object (String) | No | Product sub-category group description. |
| `category_product_code` | Object (String) | No | Product category system code. |

### 2.4. Physical Store Dimension (`dim_stores`)
* **Primary Key:** `store_key` (Surrogate Key)
* *Note: Excludes non-physical store locations (Website/Ecommerce).*

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `store_key` | Int64 | No | Sequential surrogate key generated during modeling. |
| `store_code` | Int64 | No | Unique physical store location code. |
| `store_region` | Object (String) | Yes | Province/City location formatted with Title Case. |
| `store_district` | Object (String) | Yes | District location formatted with Title Case. |

### 2.5. Date Dimension (`dim_date`)
* **Primary Key:** `date_key` (YYYYMMDD)

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `date_key` | Int64 | No | Date integer key formatted as YYYYMMDD (e.g., `20241231`). |
| `fully_date` | DateTime64 | No | Date timestamp at midnight. |
| `year` | Int32 | No | Year of transaction. |
| `quarter` | Int32 | No | Calendar Quarter (1, 2, 3, 4). |
| `month` | Int32 | No | Calendar Month (1 to 12). |
| `day` | Int32 | No | Calendar Day of the month (1 to 31). |
| `weekday` | Object (String) | No | Full name of the day of the week (e.g., `Wednesday`). |

### 2.6. Order Status Dimension (`dim_order_status`)
* **Primary Key:** `order_status_key`

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `order_status_key` | Int64 | No | Unique identifier for order status. |
| `status_description` | Object (String) | No | Description of the transaction state (e.g., `successfully delivered`). |

### 2.7. Sales Channel Dimension (`dim_channel`)
* **Primary Key:** `channel_key`

| Column Name | Data Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `channel_key` | Int64 | No | Unique identifier for sales channel. |
| `channel_name` | Object (String) | No | Name of the sales channel (`website`, `ecommerce`, `stores`, `premium`). |