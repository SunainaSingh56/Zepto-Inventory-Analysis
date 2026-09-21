# 🛒 Zepto Inventory Analysis | SQL + Python

A data-quality-focused inventory analysis of **3,732 Zepto product records** using **MySQL, Python, and Pandas**.

The project focuses on identifying and resolving data-quality issues before performing business analysis, including duplicate product records, category inconsistencies, pricing validation, inventory availability, discounts, and category performance.

> **Core approach:** Clean the data → Validate it independently → Investigate inconsistencies → Analyze with SQL → Generate business insights.

---

## 🔎 Key Findings

| Area                                        | Finding                                        |
| ------------------------------------------- | ---------------------------------------------- |
| Raw records                                 | 3,732                                          |
| After removing invalid MRP                  | 3,731                                          |
| Final cleaned records                       | 1,675                                          |
| Repeated product names across category tags | 1,187                                          |
| Python data-quality checks                  | All core checks passed                         |
| Pricing validation                          | 409 records require business-rule confirmation |
| SQL analysis                                | CTEs, JOINs, CASE, DENSE_RANK, PARTITION BY    |

### Main Data-Quality Finding

The raw dataset contained **1,187 product names appearing across multiple category tags**, which required investigation before category-level analysis.

After cleaning and deduplication, the dataset was reduced from **3,731 to 1,675 records**.

---

## 🛠️ Tools & Skills

* **MySQL 8.0** — Data cleaning, analysis, CTEs, JOINs, window functions
* **Python** — Data validation and quality checks
* **Pandas** — Data profiling and validation
* **Jupyter Notebook** — Reproducible validation workflow
* **Google Gemini API** — Supporting interpretation of data-quality findings

---

## 📊 Dataset

**Source:** Kaggle — Zepto Inventory Dataset

The original dataset contained product-level inventory information such as:

* SKU ID
* Category
* Product name
* MRP
* Discount percentage
* Available quantity
* Discounted selling price
* Weight
* Out-of-stock status

### Data Transformation

| Stage          | Records |
| -------------- | ------: |
| Raw dataset    |   3,732 |
| Remove MRP = 0 |   3,731 |
| Deduplication  |   1,675 |

The source prices were stored in **paise**, so price fields were converted to **rupees** during preparation.

### Final Dataset Schema

```text
sku_id
category
name
mrp
discountPercent
availableQuantity
discountedSellingPrice
weightInGms
outOfStock
```

---

## 🧹 Data Cleaning & Deduplication

Before business analysis, the raw data was investigated for structural and quality issues.

### 1. Invalid MRP

One record had an MRP of `0`, so it was removed before further analysis.

### 2. Category Duplication Investigation

A diagnostic query grouped products by name and counted their distinct category tags.

This identified **1,187 product names appearing across multiple category tags**.

This mattered because category-level analysis could otherwise produce misleading results.

### 3. Deduplication

After investigating the repeated records, duplicate product/category combinations were removed using SQL window-function logic.

The cleaned dataset contained **1,675 records**.

The complete cleaning queries are available in:

```text
sql/zepto_analysis.sql
```

---

## 🐍 Python Data Quality Validation

After SQL cleaning, Python/Pandas was used as an independent validation layer.

### Validation Results

| Check                   | Result |
| ----------------------- | -----: |
| Missing values          |      0 |
| Duplicate rows          |      0 |
| Duplicate SKU IDs       |      0 |
| Invalid MRP             |      0 |
| Invalid discount values |      0 |
| Negative stock          |      0 |
| Invalid selling price   |      0 |
| Negative weight         |      0 |

This provided an independent check that the cleaned dataset satisfied the defined validation rules.

Notebook:

```text
notebooks/zepto_data_quality.ipynb
```

---

## 💰 Pricing Consistency Investigation

The discounted selling price was investigated using:

```text
Expected Price = MRP × (1 - Discount% / 100)
```

### Validation Results

* **873 records** had a difference greater than ₹0.01 from the calculated price.
* A second validation using rounded discount values reconciled **1,266 records**.
* **409 records remained unresolved**.
* The average remaining difference was approximately **₹1.51**.
* The maximum difference was **₹8.00**.

The unresolved records were **not automatically classified as errors**, because the difference could result from source-system pricing rules, rounding, or other business logic.

> **Analytical principle:** A data-quality exception should be investigated before being classified as an error.

---

## 🤖 AI-Assisted Data Quality Interpretation

Google Gemini API was used as a **supporting interpretation layer** after the deterministic Python validation.

### Workflow

```text
Python Validation
       ↓
Identify Exceptions
       ↓
Gemini Interpretation
       ↓
Business-Rule Review
```

Python remained the **source of truth for validation**.

Gemini was used to help interpret:

* Potential causes of pricing discrepancies
* Data-quality patterns
* Possible business impact
* Validation rules requiring further investigation

> **Python validates → SQL analyzes → Gemini supports interpretation → Business rules confirm**

The AI assessment is saved in:

```text
ai_quality_assessment.md
```

---

# 📈 SQL Business Analysis

Once the data was cleaned and validated, SQL was used to generate business-oriented insights.

---

## 1. High-MRP Products That Are Out of Stock

Identifies expensive products currently marked as unavailable.

```sql
SELECT
    name,
    category,
    mrp,
    availableQuantity,
    outOfStock
FROM zepto
WHERE outOfStock = 1
ORDER BY mrp DESC;
```

**Business use:** Helps identify higher-value products that may require inventory attention.

---

## 2. Estimated Inventory Value

Inventory value was estimated using:

```text
MRP × Available Quantity
```

```sql
SELECT
    category,
    ROUND(SUM(mrp * availableQuantity), 2) AS estimated_inventory_value
FROM zepto
GROUP BY category
ORDER BY estimated_inventory_value DESC;
```

> **Important:** This represents an estimated inventory value based on available quantity and MRP. It is **not actual sales revenue**.

---

## 3. Top 3 Discounted Products by Category

A window function was used to rank products within each category.

```sql
WITH ranked_products AS (
    SELECT
        name,
        category,
        discountPercent,
        DENSE_RANK() OVER (
            PARTITION BY category
            ORDER BY discountPercent DESC
        ) AS discount_rank
    FROM zepto
)
SELECT
    name,
    category,
    discountPercent
FROM ranked_products
WHERE discount_rank <= 3;
```

**SQL concepts:** `CTE`, `DENSE_RANK()`, `PARTITION BY`

---

## 4. Category Performance Analysis

Category-level aggregation was used to examine:

* Product count
* Available quantity
* Out-of-stock products
* Average discount
* Estimated inventory value

Ranking logic was then applied to compare category performance.

---

# 🔗 JOIN-Based Operational Analysis

A supplementary `category_info` reference table was used to demonstrate how inventory data could be combined with operational information such as:

* Category manager
* Warehouse zone
* Reorder threshold

> **Note:** These operational fields are supplementary analytical data and are **not part of the original Kaggle dataset**.

Example:

```sql
SELECT
    z.category,
    c.category_manager,
    c.warehouse_zone,
    SUM(z.availableQuantity) AS total_stock,
    SUM(
        CASE
            WHEN z.availableQuantity <= c.reorder_threshold
            THEN 1
            ELSE 0
        END
    ) AS products_needing_reorder
FROM zepto z
INNER JOIN category_info c
    ON z.category = c.category
GROUP BY
    z.category,
    c.category_manager,
    c.warehouse_zone;
```

This demonstrates how cleaned inventory data can be connected with operational reference data to support potential reorder analysis.

---

# 💡 Business Takeaways

### Inventory & Category Analysis

* Data-quality issues should be resolved before using category-level metrics.
* Cooking Essentials showed substantially higher estimated inventory value than Fruits & Vegetables in the cleaned analysis.
* Fruits & Vegetables had an average discount of approximately **15.93%**.
* Out-of-stock and inventory-value analysis can help identify areas requiring operational attention.

### Pricing

The pricing validation demonstrated that formula mismatches should not automatically be treated as data errors. Additional business rules may be required to explain the remaining **409 unresolved records**.

---

# ⚠️ Data Limitations

This analysis is based on a **product-level inventory snapshot**.

The source dataset does not provide:

* Historical inventory levels
* Customer orders
* Actual sales transactions
* Sales velocity
* Complete warehouse-level operational data

Therefore:

* Inventory-value calculations are **estimates**, not actual revenue.
* Historical inventory trends cannot be calculated.
* Sales velocity cannot be measured from the source data.
* Reorder analysis depends on supplementary operational reference data and business rules.

---

# 📁 Project Structure

```text
Zepto-Inventory-Analysis/
│
├── data/
│   ├── zepto_clean.csv
│   └── zepto_v2.csv
│
├── notebooks/
│   ├── zepto_data_quality.ipynb
│   └── export_clean_data.ipynb
│
├── sql/
│   ├── zepto_db_setup.sql
│   └── zepto_analysis.sql
│
├── ai_quality_assessment.md
│
├── screenshots/
│
└── README.md
```

---

# 🚀 How to Run

## 1. Clone the Repository

```bash
git clone https://github.com/SunainaSingh56/Zepto-Inventory-Analysis.git
cd Zepto-Inventory-Analysis
```

## 2. Set Up MySQL

Run:

```text
sql/zepto_db_setup.sql
```

This creates the required database/table structure and loads the cleaned data.

## 3. Run SQL Analysis

Open:

```text
sql/zepto_analysis.sql
```

Execute the queries in MySQL 8.0.

## 4. Run Python Validation

Open:

```text
notebooks/zepto_data_quality.ipynb
```

Run the notebook to reproduce the data-quality checks.

## 5. Gemini API

If reproducing the AI-assisted interpretation, configure your Gemini API key as an environment variable.

**Never commit API keys or other credentials to GitHub.**

---

# 🔮 Future Improvements

* Add historical inventory/sales data for trend analysis
* Add stockout and sales-velocity analysis
* Build a Power BI dashboard for interactive monitoring

---

## 📌 Project Workflow

```text
Raw Data
   ↓
Data Cleaning
   ↓
Deduplication
   ↓
Python Validation
   ↓
Pricing Investigation
   ↓
SQL Analysis
   ↓
Business Insights
```

### Main Learning

The key focus of this project was not simply writing SQL queries. It was understanding that **data quality comes before business analysis**.

The workflow demonstrates how a Data Analyst can:

**Discover → Investigate → Clean → Validate → Analyze → Interpret**
