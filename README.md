# ITB Commercial Performance & Strategic Growth Analytics
*An End-to-End Business Intelligence, Star Schema Modeling & Financial Simulation Project (BI9 Competition - Rounds 2 & 3)*

---

## 📌 1. Business Problem & Context
**ITB** is a commercial enterprise distributing multiple consumer lines across Vietnam. Despite achieving stellar top-line financial growth, the brand's executive board is facing a critical strategic bottleneck: **The Empty Growth Syndrome**. 

While overall revenues are climbing, the customer foundation is silently decaying under extremely high churn rates. Historically, ITB executed a "Bait-and-Funnel" strategy—using low-cost Products A & B as customer acquisition hooks to attract mass-market users, with the expectation of subsequently upgrading them to the premium, cash-cow Product C. 

This project was established to audit this funnel, model a scalable Star Schema Data Warehouse, and execute forward-looking financial simulations to restructure ITB's business model.

### Key Objectives:
1. **Audit Funnel Efficiency**: Chronologically evaluate whether Products A & B act as successful transition conduits to the premium Product C.
2. **Diagnose the Retention Crisis**: Quantify customer decay through quarterly cohort analysis to identify exactly where the "leaky bucket" is occurring.
3. **Build a Star Schema Data Warehouse**: Transition the flat, messy transactional database into an optimized, normalized dimensional schema ready for BI ingestion.
4. **Model Forward-Looking Strategic Simulations**: Integrate the new 2025 Website cost structure, calculate Break-even Points (BEP), and simulate 2026 cash-flow reallocations to scale net profits.

> 📄 **Official Competition Prompts & Guidelines**: You can review the official round requirements and questions in the [Round 02 Requirements](docs/bi9-round2-requirements.pdf) and [Round 03 Requirements](docs/bi9-round3-requirements.pdf).

---

## 🛠️ 2. Analytical Approach & Workflow
To address these multi-layered challenges, we implemented an end-to-end analytics framework spanning data engineering, diagnostics, schema modeling, and corporate finance forecasting:

[Raw transactional data (5 parts)] ➔ [ETL Data Pipeline & Cleaning] ➔ [Diagnostic Business Analytics] ➔ [Star Schema Warehousing] ➔ [Financial Re-Costing & BEP (2025)] ➔ [Strategic Growth Simulations (2026)]

* **Environment**: VS Code (Jupyter Extension)
* **Stack**: Python (pandas, numpy, scikit-learn, matplotlib, seaborn, plotly, adjustText) & Power BI.

---

## 🔍 3. Data Engineering & ETL Pipeline (Phase 1)
The raw transactional database contained over **2.1 million records** across 5 partitioned files. In **`notebooks/01_etl_data_pipeline.ipynb`**, we resolved the following issues to build our single source of truth:

1. **Date & Type Standardization**: Preserved correct formats by parsing `order_date` and `dob` into proper `datetime64[ns]` types.
2. **Categorical Normalization**: Consolidated redundant education levels (e.g., merging "Bachelor's degree" into "Bachelor") and mapped missing categorical values to `'Unknown'`.
3. **Preserving Referential Integrity**: Discovered and removed **56,435 orphan transactional records** (referencing customer IDs like `-43` that did not exist in the customer dimension), establishing a clean transaction database of **2,051,208 records**.
4. **Excluding Sparsity Bias**: Excluded the year **2021** from the analytical master dataset because it only recorded a single day of transactional history, which would have heavily skewed YoY trend analyses.

---

## 📊 4. Diagnostic Analytics: Funnel & Retention Crisis (Phase 2)
In **`notebooks/02_diagnostic_business_analytics.ipynb`**, we quantified the operational bottlenecks of ITB:

### 4.1. The Growth Paradox ("Empty Growth")
Our analysis verified a striking mismatch. While revenue grew by **21.2%** from 2022 to 2024, the active customer count plummeted by **-10.57%** in 2023. Top-line growth was artificially sustained by extracting more transactional value from a shrinking, non-recurrent customer base.

| Year | Annual Revenue (VND) | Gross Profit (VND) | Active Customer Count | Customer YoY Growth (%) |
| :---: | :---: | :---: | :---: | :---: |
| **2022** | 7,213,973,112,382 | 698,891,680,305 | 406,738 | Baseline |
| **2023** | 7,097,189,459,768 | 782,153,968,986 | 363,732 | **-10.57%** |
| **2024** | 8,746,644,109,265 | 760,291,970,111 | 383,421 | +5.41% |

### 4.2. Complete Collapse of the Bait Funnel
The transition "bridge" failed. The volume of new customers acquired via the bait Products A & B fell by **59%**, and their subsequent upgrade rate to the core Product C collapsed from **13.30% in 2022 to an insignificant 2.56% in 2024**.

<p align="center">
  <img src="assets/charts/bait_funnel_conversion.png" width="85%" />
</p>
<p align="center"><i>Figure 4.1: Dramatic shrinkage and leakage of the historical customer acquisition funnel</i></p>

### 4.3. Customer Retention Crisis
ITB operates with an alarmingly high overall **90-day churn rate of 85.01%**. Cohort analysis shows a steep drop immediately after the first quarter, with **86% to 91% of customers never returning** to make a second purchase.

<p align="center">
  <img src="assets/charts/cohort_retention_heatmap.png" width="95%" />
</p>
<p align="center"><i>Figure 4.2: Quarterly cohort retention heatmap highlighting the immediate drop-off after Quarter 0</i></p>

---

## 🔮 5. Customer Value Profiling & RFM Segmentation
By segmenting customers into those who bought Product C vs. those who only bought bait products, we proved that Product C buyers are highly engaged and valuable. On average, Product C buyers exhibit **double the purchase frequency** and spend **over 5.3 times more** than other customers.

<p align="center">
  <img src="assets/charts/rfm_comparison.png" width="100%" />
</p>
<p align="center"><i>Figure 5.1: Comparative RFM metrics proving Product C buyers represent a premium "Golden Segment"</i></p>

---

## 🔒 6. Star Schema Data Warehouse Architecture (Phase 3)
In **`notebooks/03_star_schema_data_modeling.ipynb`**, the flat master database was modeled into an optimized, standardized Star Schema consisting of **6 Dimension Tables** and **1 Central Fact Table**:

```text
               [dim_customer]           [dim_product]
              (customer_key)            (product_key)
                     │                       │
                     └───► [fact_order] ◄────┘
                     ┌───►   (Central)  ◄────┐
                     │                       │
                [dim_stores]            [dim_channel]
                (store_key)             (channel_key)
                     │                       │
                     ▼                       ▼
                [dim_date]           [dim_order_status]
                (date_key)          (order_status_key)
```
* **Business Logic**: Surrogate keys were generated to replace natural business keys, ensuring seamless indexing, referential integrity, and high-performance querying in BI tools. All dimensional datasets (`dim_*.csv`) and transaction facts (`fact_order.csv`) are exported into the `data/processed/` directory.

---

## 🖥️ 7. Interactive Business Intelligence Dashboard

### 7.1. Executive Performance Dashboard
An interactive executive dashboard was constructed in Power BI to enable C-suite monitoring of core metrics: revenue, net profits, channel margins, and product sales. 

![ITB Executive Dashboard Preview](assets/dashboards/dashboard_preview.png)

### 7.2. Channel Margin & Fixed Cost Mismatch
The analytics dashboard highlights a massive cost structural mismatch. Physical channels (Store and Premium) are heavily burdened by high retail staff salaries, costing **838.6 Billion VND annually** and dragging net profit margins down to **48.8% - 58.1%**. In contrast, online channels (Website and Ecommerce) represent highly streamlined models, with the Website channel delivering a superior **95.36% net profit margin**.

<p align="center">
  <img src="assets/charts/channel_performance_margin.png" width="85%" />
</p>
<p align="center"><i>Figure 7.1: Net profit margin trend by channel, showing the superior profitability of online models</i></p>

---

## 💡 8. Forward-Looking Strategic Simulations (Phase 4)
In **`notebooks/04_strategic_financial_simulations.ipynb`**, we modeled financial forecasts for 2025 and 2026.

### 8.1. The 130% BEP Target Paradox (2025)
Using the new **Website Fee 2025** cost parameters (Fixed Cost: 20.7B VND/year; Variable: PG fee, email marketing, shipping), we calculated:
*   Website Average Contribution Margin: **10,648,159 VND / order**
*   Annual Break-even Point (BEP): **1,944 orders / year**
*   **The Trap:** Targeting 130% BEP (2,527 orders/year) in 2025 would curtail Website transaction volume by **98.6%**, causing a severe **-25.61% revenue drop** at the corporate level. This proves that BEP should only be monitored as a safety net, not targeted as a commercial ceiling.

<p align="center">
  <img src="assets/charts/revenue_projection_130_bep.png" width="75%" />
</p>
<p align="center"><i>Figure 8.1: Negative corporate revenue impact if the Website channel is constrained to 130% BEP</i></p>

### 8.2. Strategy "A+" Financial Projections (2026)
We simulated **Strategy A+**—a comprehensive plan that phases out the low-performing Product B, specializes store layouts, renegotiates partner commissions, and reallocates **30% of Stores fixed cost** into Website marketing. 

Assuming a digital ROAS of 5:1 (Base Case) and 6:1 (Optimistic Case), this reallocation model is projected to increase corporate net profits by **+6.85% (+418B VND)** and **+11.34% (+692B VND)** respectively.

<p align="center">
  <img src="assets/charts/strategy_aplus_comparison.png" width="80%" />
</p>
<p align="center"><i>Figure 8.2: 2026 projected corporate net profit increases under Strategy A+ simulations</i></p>

---

## 🚀 9. Actionable Business Recommendations
1. **Dismantle the Legacy Bait Strategy:** Discontinue Product B immediately and transition Product A from an acquisition hook into a self-sustaining cash cow.
2. **Reallocate Capital to Digital Channels:** Pivot marketing budgets from high-overhead physical retail stores to direct Website acquisition.
3. **Establish a Dedicated CRM System:** Focus on retention campaigns and loyalty tiers for Product C buyers to capture their high Customer Lifetime Value (CLV) and resolve the churn crisis.

---

## 📁 10. Repository Structure & Execution
```text
BI9-R2-R3-ITB-Analytics/
├── assets/                     # Diagnostic charts and dashboard previews
│   ├── charts/                 # Generated matplotlib/seaborn charts
│   └── dashboards/             # Screenshots of Power BI dashboard
├── data/                       # Local database folder (Excluded from Git control)
│   ├── raw/                    # Raw source CSV files
│   └── processed/              # Cleaned master & Star Schema files
├── docs/                       # Project guidelines and prompts
├── notebooks/                  # Step-by-step ETL and analytics pipeline
│   ├── 01_etl_data_pipeline.ipynb
│   ├── 02_diagnostic_business_analytics.ipynb
│   ├── 03_star_schema_data_modeling.ipynb
│   └── 04_strategic_financial_simulations.ipynb
├── requirements.txt            # Python dependencies
├── data_dictionary.md          # Comprehensive data schema dictionary
└── README.md                   # Project presentation