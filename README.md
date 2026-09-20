# 🛒 Zepto Inventory Analysis | SQL + Python + AI-Assisted Data Quality

An end-to-end **data analytics project** focused on cleaning, validating, and analyzing Zepto inventory data using **MySQL, Python, Pandas, and Gemini AI**.

The project follows a practical analytics workflow:

**Raw Data → Data Cleaning → Deduplication → Data Quality Validation → SQL Analysis → Business Insights**

> **Key principle:** Before using data for business decisions, first make sure the data can be trusted.

---

## 📌 Table of Contents

* [🎯 Project Objective](#-project-objective)
* [🛠️ Tools Used](#️-tools-used)
* [📊 Dataset](#-dataset)
* [📁 Project Structure](#-project-structure)
* [🔄 Analysis Workflow](#-analysis-workflow)
* [📋 Business Questions](#-business-questions)
* [🧹 Data Cleaning & Deduplication](#-data-cleaning--deduplication)
* [🐍 Python Data Quality Validation](#-python-data-quality-validation)
* [🤖 AI-Assisted Data Quality Analysis](#-ai-assisted-data-quality-analysis)
* [🔎 Interactive SQL Analysis](#-interactive-sql-analysis)
* [💡 Key Insights](#-key-insights)
* [⚠️ Challenges](#️-challenges)
* [▶️ How to Run](#️-how-to-run)
* [🚀 What I Would Do Next](#-what-i-would-do-next)
* [📂 Project Files](#-project-files)

---

## 🎯 Project Objective

The objective was to analyze Zepto inventory data while treating **data quality as the first step of the analysis**.

The project focuses on:

* Identifying and removing invalid records
* Investigating duplicate product/category records
* Validating the cleaned dataset using Python
* Checking pricing consistency
* Analyzing revenue, discounts, stock availability, and inventory
* Using SQL techniques such as **CTEs, window functions, CASE statements, and JOINs**
* Using Gemini AI to interpret validated data-quality findings

---

## 🛠️ Tools Used

| Tool                  | Purpose                                                    |
| --------------------- | ---------------------------------------------------------- |
| **MySQL 8.0**         | Data cleaning, SQL analysis, JOINs, CTEs, window functions |
| **Python**            | Data-quality validation                                    |
| **Pandas**            | Data inspection and validation                             |
| **Jupyter Notebook**  | Python-based analysis                                      |
| **Google Gemini API** | AI-assisted interpretation of validated findings           |
| **VS Code**           | Development environment                                    |

---

## 📊 Dataset

**Source:** Kaggle — Zepto Inventory Dataset

### Dataset Transformation

| Stage                                |      Rows |
| ------------------------------------ | --------: |
| Raw dataset                          | **3,732** |
| After removing invalid `MRP = 0` row | **3,731** |
| After deduplication                  | **1,675** |
| Final columns                        |     **9** |

### Final Dataset Schema

| Column                   | Description                  |
| ------------------------ | ---------------------------- |
| `sku_id`                 | Unique SKU identifier        |
| `category`               | Product category             |
| `name`                   | Product name                 |
| `mrp`                    | Maximum Retail Price         |
| `discountPercent`        | Discount percentage          |
| `availableQuantity`      | Available inventory quantity |
| `discountedSellingPrice` | Selling price after discount |
| `weightInGms`            | Product weight               |
| `outOfStock`             | Stock availability flag      |

> **Note:** Prices in the source data were stored in **paise** and converted to **rupees** during preprocessing.

---

## 📁 Project Structure

```text
ZEPTO SQL PROJECT/
│
├── data/
│   └── zepto_clean.csv
│
├── python/
│   └── zepto_data_quality.ipynb
│
├── sql/
│   ├── zepto_analysis.sql
│   └── zepto_db_setup.sql
│
├── docs/
│   └── ai_quality_assessment.md
│
├── screenshots/
│   ├── sql_avg_discount_category.png
│   ├── sql_dedup_result.png
│   ├── sql_join_query_output.png
│   ├── sql_outofstock_highmrp.png
│   ├── sql_revenue_by_category.png
│   └── sql_schema_overview.png
│
└── README.md
```

---

## 🔄 Analysis Workflow

```text
Raw Kaggle Dataset
        ↓
Initial Data Exploration
        ↓
Remove Invalid MRP = 0
        ↓
Investigate Duplicate Product/Category Records
        ↓
Deduplicate Dataset
        ↓
Export Clean Dataset
        ↓
Python Data Quality Validation
        ↓
Pricing Consistency Investigation
        ↓
AI-Assisted Interpretation
        ↓
SQL Business Analysis
        ↓
Business Insights
```

---

## 📋 Business Questions

The SQL analysis was designed around practical inventory and commercial questions:

1. Which products have the highest discounts?
2. Which high-MRP products are out of stock?
3. Which categories have the highest estimated revenue?
4. Which categories offer the highest average discounts?
5. Which products represent the strongest deals?
6. How does inventory vary by category?
7. Which categories require attention based on stock thresholds?
8. How can category-level information be combined with inventory data?
9. Which categories perform strongly within each warehouse zone?
10. What patterns can be identified from the cleaned dataset?

---

# 🧹 Data Cleaning & Deduplication

## 1️⃣ Removing Invalid MRP

The raw dataset contained one record where:

```text
MRP = 0
```

This record was removed because a zero MRP is not meaningful for the pricing analysis.

---

## 2️⃣ Investigating Duplicate Product Records

During the initial analysis, some product names appeared under multiple category tags.

A diagnostic query was used to identify these cases:

```sql
SELECT
    name,
    COUNT(DISTINCT category) AS category_count
FROM zepto1
GROUP BY name
HAVING category_count > 1
ORDER BY category_count DESC;
```

The investigation identified **1,187 products appearing across multiple category tags**.

This was important because the initial category-level revenue results contained suspiciously similar totals across unrelated categories.

Instead of immediately using those results, the data was investigated first.

---

## 3️⃣ Deduplication

After investigating the duplicate records, the dataset was deduplicated before performing the final business analysis.

### Result

```text
3,731 records
      ↓
1,675 records
```

![Deduplication Result](sql_dedup_result.png)

> **Key takeaway:** Data validation changed the dataset before business insights were generated. This helped prevent potentially misleading conclusions from duplicated records.

---

# 🐍 Python Data Quality Validation

After SQL cleaning, the final dataset was validated independently using **Python + Pandas**.

Notebook:

`python/zepto_data_quality.ipynb`

### Validation Results

| Validation               | Result |
| ------------------------ | -----: |
| Missing values           |  **0** |
| Duplicate rows           |  **0** |
| Duplicate SKU IDs        |  **0** |
| Invalid MRP values       |  **0** |
| Discount outside 0–100%  |  **0** |
| Negative stock values    |  **0** |
| Invalid selling prices   |  **0** |
| Negative product weights |  **0** |

### Data Types

The final dataset contains:

* Integer columns for IDs and quantities
* Float columns for prices and discounts
* Object columns for category/product information
* Boolean values for stock availability

---

## 💰 Pricing Consistency Check

The expected discounted price was calculated using:

```python
expected_price = mrp * (1 - discountPercent / 100)
```

The initial comparison identified:

```text
873 records
```

where the stored selling price did not match the simple formula within ₹0.01.

A second validation using a rounded discount calculation found:

```text
1,266 records reconciled
```

The remaining:

```text
409 records
```

did not reconcile under either calculation and therefore require confirmation of the actual pricing/business rule used by the source system.

### Important distinction

These 409 records are **not automatically treated as pricing errors**.

The analysis identifies them as **records requiring business-rule confirmation**.

---

# 🤖 AI-Assisted Data Quality Analysis

Gemini was used as a **supporting interpretation layer**, not as the source of truth.

### Workflow

```text
Python Validation
      ↓
Validated Findings
      ↓
Gemini Interpretation
      ↓
Classification + Business Impact
      ↓
Recommended Validation Rules
```

Python was responsible for deterministic checks such as:

* Missing values
* Duplicate records
* Duplicate SKU IDs
* Invalid ranges
* Pricing discrepancies

Gemini was then used to:

* Interpret validated findings
* Classify observations
* Explain potential business impact
* Suggest validation rules
* Highlight areas requiring business-rule confirmation

> **Python remained the source of truth. AI was used to interpret findings rather than replace analytical logic.**

📄 [View AI Quality Assessment](docs/ai_quality_assessment.md)

---

# 🔎 Interactive SQL Analysis

All detailed SQL queries are available in:

`sql/zepto_analysis.sql`

The sections below provide selected queries using GitHub's expandable `<details>` functionality.

---

## 1️⃣ Data Exploration

<details>
<summary><b>Q1 — Top 10 Most-Discounted Products</b></summary>

### Query

```sql
SELECT DISTINCT
    name,
    mrp,
    discountPercent
FROM zepto_clean
ORDER BY discountPercent DESC
LIMIT 10;
```

### Purpose

Identify products receiving the highest percentage discounts.

</details>

---

<details>
<summary><b>Q2 — High-MRP Products That Are Out of Stock</b></summary>

### Query

```sql
SELECT DISTINCT
    name,
    mrp
FROM zepto_clean
WHERE outOfStock = 'TRUE'
  AND mrp > 300
ORDER BY mrp DESC;
```

### Purpose

Identify higher-priced products that are currently marked as out of stock.

![High MRP Out of Stock](sql_outofstock_highmrp.png)

</details>

---

## 2️⃣ Category-Level Analysis

<details>
<summary><b>Q3 — Estimated Revenue by Category</b></summary>

### Query

```sql
SELECT
    category,
    ROUND(
        SUM(discountedSellingPrice * availableQuantity),
        2
    ) AS estimated_revenue
FROM zepto_clean
GROUP BY category
ORDER BY estimated_revenue DESC;
```

### Purpose

Estimate the inventory value represented by each category using:

```text
Discounted Selling Price × Available Quantity
```

![Revenue by Category](sql_revenue_by_category.png)

</details>

---

<details>
<summary><b>Q4 — Average Discount by Category</b></summary>

### Query

```sql
SELECT
    category,
    ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto_clean
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

### Finding

**Fruits & Vegetables** recorded an average discount of approximately **15.93%** in the analyzed dataset.

![Average Discount by Category](sql_avg_discount_category.png)

</details>

---

## 3️⃣ Advanced SQL Analysis

<details>
<summary><b>Q5 — Top 3 Discounted Products per Category</b></summary>

### Query

```sql
WITH category_ranked AS (
    SELECT
        name,
        category,
        discountPercent,
        DENSE_RANK() OVER (
            PARTITION BY category
            ORDER BY discountPercent DESC
        ) AS rnk
    FROM zepto_clean
)
SELECT
    name,
    category,
    discountPercent,
    rnk
FROM category_ranked
WHERE rnk <= 3
ORDER BY category, rnk;
```

### SQL Concepts Used

* CTE
* `DENSE_RANK()`
* `PARTITION BY`
* Category-level ranking

</details>

---

<details>
<summary><b>Q6 — Category Performance Summary</b></summary>

### Query

```sql
WITH category_summary AS (
    SELECT
        category,
        COUNT(DISTINCT name) AS unique_products,
        ROUND(AVG(discountPercent), 2) AS avg_discount_pct,
        ROUND(AVG(mrp), 2) AS avg_mrp,
        ROUND(
            SUM(discountedSellingPrice * availableQuantity),
            2
        ) AS total_revenue,
        SUM(
            CASE
                WHEN outOfStock = 'TRUE' THEN 1
                ELSE 0
            END
        ) AS out_of_stock_count,
        SUM(
            CASE
                WHEN outOfStock = 'FALSE' THEN 1
                ELSE 0
            END
        ) AS in_stock_count
    FROM zepto_clean
    GROUP BY category
)
SELECT
    *,
    RANK() OVER (
        ORDER BY total_revenue DESC
    ) AS revenue_rank
FROM category_summary
ORDER BY revenue_rank;
```

### SQL Concepts Used

* CTE
* Aggregation
* `CASE WHEN`
* `RANK()`
* Multiple business metrics

</details>

---

# 🔗 Multi-Table JOIN Analysis

A reference table named `category_info` was used to demonstrate multi-table analysis.

It contains category-level fields such as:

* Category
* Category manager
* Warehouse zone
* Reorder threshold

This table supports additional JOIN-based analysis.

![SQL JOIN Output](sql_join_query_output.png)

---

<details>
<summary><b>Q7 — Category Revenue with Reference Data</b></summary>

```sql
SELECT
    ci.category,
    ci.category_manager,
    ci.warehouse_zone,
    ROUND(
        SUM(
            z.discountedSellingPrice *
            z.availableQuantity
        ),
        2
    ) AS total_revenue
FROM zepto_clean z
INNER JOIN category_info ci
    ON z.category = ci.category
GROUP BY
    ci.category,
    ci.category_manager,
    ci.warehouse_zone
ORDER BY total_revenue DESC;
```

### Concepts Used

* `INNER JOIN`
* Aggregation
* Multiple-table analysis

</details>

---

<details>
<summary><b>Q8 — Identifying Categories Without Matching Reference Data</b></summary>

```sql
SELECT
    z.category,
    ci.category_manager,
    ci.warehouse_zone,
    COUNT(z.sku_id) AS total_skus,
    ROUND(
        AVG(z.discountPercent),
        2
    ) AS avg_discount
FROM zepto_clean z
LEFT JOIN category_info ci
    ON z.category = ci.category
GROUP BY
    z.category,
    ci.category_manager,
    ci.warehouse_zone
ORDER BY total_skus DESC;
```

### Purpose

A `LEFT JOIN` keeps all categories from the inventory dataset and helps identify categories without matching reference information.

</details>

---

<details>
<summary><b>Q9 — Stock Status Using CASE WHEN</b></summary>

```sql
SELECT
    ci.category,
    ci.category_manager,
    ci.warehouse_zone,
    ci.reorder_threshold,
    SUM(z.availableQuantity) AS current_inventory,
    CASE
        WHEN SUM(z.availableQuantity)
             < ci.reorder_threshold
            THEN 'RESTOCK NOW'

        WHEN SUM(z.availableQuantity)
             < ci.reorder_threshold * 1.5
            THEN 'LOW STOCK'

        ELSE 'SUFFICIENT'
    END AS stock_status
FROM zepto_clean z
INNER JOIN category_info ci
    ON z.category = ci.category
GROUP BY
    ci.category,
    ci.category_manager,
    ci.warehouse_zone,
    ci.reorder_threshold
ORDER BY current_inventory ASC;
```

### SQL Concepts Used

* `INNER JOIN`
* `SUM()`
* `CASE WHEN`
* Business-rule logic
* Inventory threshold analysis

</details>

---

<details>
<summary><b>Q10 — Revenue Ranking Within Warehouse Zones</b></summary>

```sql
WITH zone_revenue AS (
    SELECT
        ci.warehouse_zone,
        ci.category,
        ci.category_manager,
        ROUND(
            SUM(
                z.discountedSellingPrice *
                z.availableQuantity
            ),
            2
        ) AS revenue
    FROM zepto_clean z
    INNER JOIN category_info ci
        ON z.category = ci.category
    GROUP BY
        ci.warehouse_zone,
        ci.category,
        ci.category_manager
)
SELECT
    warehouse_zone,
    category,
    category_manager,
    revenue,
    RANK() OVER (
        PARTITION BY warehouse_zone
        ORDER BY revenue DESC
    ) AS rank_within_zone
FROM zone_revenue
ORDER BY
    warehouse_zone,
    rank_within_zone;
```

### SQL Concepts Used

* CTE
* JOIN
* Aggregation
* Window function
* `PARTITION BY`
* Ranking within groups

</details>

---

# 💡 Key Insights

### 1. Data quality changed the analytical dataset

The dataset reduced from:

**3,732 → 3,731 → 1,675 records**

after invalid-value removal and deduplication.

This demonstrated that cleaning decisions can materially affect downstream business analysis.

---

### 2. Duplicate-category records required investigation

The presence of **1,187 products across multiple category tags** created suspicious category-level results during the initial analysis.

Rather than accepting those results, the duplication pattern was investigated before continuing.

---

### 3. The cleaned dataset passed core structural checks

Python validation found:

* **0 missing values**
* **0 duplicate rows**
* **0 duplicate SKU IDs**
* **0 invalid MRP values**
* **0 invalid discount ranges**
* **0 negative inventory quantities**
* **0 invalid selling prices**
* **0 negative weights**

---

### 4. Pricing logic requires business-rule confirmation

The pricing validation showed that:

* **1,266 records** reconciled under a rounded discount calculation.
* **409 records** still require confirmation of the source pricing rule.

This demonstrates why a mathematical mismatch should not automatically be classified as a business error without understanding the underlying pricing logic.

---

### 5. SQL can turn cleaned inventory data into business analysis

The project demonstrates SQL techniques including:

```text
GROUP BY
HAVING
CASE WHEN
CTEs
INNER JOIN
LEFT JOIN
DENSE_RANK
RANK
PARTITION BY
```

These were applied to questions around discounts, revenue, stock availability, category performance, and warehouse-zone analysis.

---

# ⚠️ Challenges

### Challenge 1 — Duplicate Data

The initial dataset contained repeated product/category records that could distort category-level analysis.

**Approach:**
Investigate the duplication pattern first, then deduplicate before generating final insights.

---

### Challenge 2 — Pricing Mismatch

The stored selling price did not always match the simplest discount formula.

**Approach:**
Test multiple deterministic pricing rules in Python instead of assuming every mismatch was an error.

---

### Challenge 3 — Separating Validation from Interpretation

AI-generated explanations can be useful but should not replace deterministic data validation.

**Approach:**

```text
Python → validates
SQL → analyzes
Gemini → interprets
Human/business rule → confirms
```

---

# ▶️ How to Run

## 1. Clone the repository

```bash
git clone https://github.com/SunainaSingh56/Zepto-Inventory-Analysis.git
```

## 2. Set up MySQL

Open:

```text
sql/zepto_db_setup.sql
```

Run the setup script in MySQL 8.0.

---

## 3. Run SQL Analysis

Open:

```text
sql/zepto_analysis.sql
```

Execute the queries in MySQL Workbench or another MySQL-compatible environment.

---

## 4. Run Python Validation

Open:

```text
python/zepto_data_quality.ipynb
```

Install the required packages if needed:

```bash
pip install pandas jupyter
```

Then run the notebook.

---

## 5. Gemini API

If reproducing the AI-assisted analysis, configure the API key as an environment variable:

```text
GEMINI_API_KEY
```

> Never commit API keys or credentials to GitHub.

---

# 🚀 What I Would Do Next

If more operational data were available, the project could be extended with:

* Historical inventory trends
* Daily stock movement
* Sales velocity
* Reorder-point optimization
* Stockout rate by category
* Discount vs. sales analysis
* Price-change monitoring
* Automated data-quality checks
* Power BI dashboard for business users

---

# 📂 Project Files

| File                              | Purpose                            |
| --------------------------------- | ---------------------------------- |
| `data/zepto_clean.csv`            | Final cleaned dataset              |
| `python/zepto_data_quality.ipynb` | Data-quality validation            |
| `sql/zepto_db_setup.sql`          | Database/table setup               |
| `sql/zepto_analysis.sql`          | SQL business analysis              |
| `docs/ai_quality_assessment.md`   | Gemini-assisted quality assessment |
| `screenshots/`                    | SQL output screenshots             |

---

## 📌 Key Takeaway

This project was not only about writing SQL queries.

It demonstrates an end-to-end analytical approach:

> **Clean the data → Validate it → Understand its limitations → Analyze it → Translate the results into business insights.**

The most important lesson from the project was that **data quality should be validated before trusting the business conclusions produced from the data.**

---

## 🔗 Project

**GitHub:**
https://github.com/SunainaSingh56/Zepto-Inventory-Analysis
