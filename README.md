# KDDM Assignment 1: Data Quality Analysis and ERD Design

## Assignment Context

This is my completed Assignment 1 for **SWEN466: Knowledge Discovery and Data Mining** at BITS College, Department of Software Engineering. The assignment was due on .

The objective was to explore the Olist e-commerce dataset using DuckDB, complete five SQL exercises analyzing data quality, and construct an entity-relationship diagram (ERD) using crow's foot notation. I validated primary keys, foreign keys, and relationship cardinalities through SQL queries rather than inferring them from column names alone.

## Dataset

I worked with the **Brazilian E-Commerce Public Dataset by Olist**, which contains real commercial data from the Brazilian e-commerce platform. The dataset includes:

- `olist_customers_dataset.csv` – 99,441 customer records
- `olist_orders_dataset.csv` – 99,441 order records
- `olist_order_items_dataset.csv` – 112,650 order line items
- `olist_products_dataset.csv` – 32,951 product records
- `olist_sellers_dataset.csv` – 3,095 seller records

All CSV files are located in the `data/` directory.

## Tools and Technologies

- **Python 3.14** – Programming language and runtime
- **Jupyter Notebook** – Interactive development environment for data exploration
- **DuckDB 1.5.6** – Embedded SQL database for analytics
- **pandas 3.0.6** – Data manipulation and analysis
- **uv** – Fast Python package installer and environment manager
- **Mermaid** – Diagram generation for the ERD

## Project Structure

```
kddm_lab_assignment/
├── data/
│   ├── olist_customers_dataset.csv      # Source: customer records
│   ├── olist_orders_dataset.csv         # Source: order records
│   ├── olist_order_items_dataset.csv    # Source: order line items
│   ├── olist_products_dataset.csv       # Source: product catalog
│   ├── olist_sellers_dataset.csv        # Source: seller information
│   ├── raw/                             # Copies loaded into DuckDB raw schema
│   └── warehouse/                       # DuckDB database file (generated)
├── DWH_OLAP_Assignment_1_Notebook.ipynb # Main notebook with all exercises
├── Olist ERD – Crow's Foot Notation (1).pdf  # Entity-relationship diagram
├── notes.md                             # Short note answering 3 questions
├── pyproject.toml                       # Project dependencies
├── uv.lock                              # Locked dependency versions
├── .python-version                      # Python version specification
├── .gitignore                           # Git ignore rules
└── README.md                            # This file
```

## Setup Instructions

### Prerequisites

- Python 3.14 or compatible version
- `uv` package manager (recommended) or `pip`

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/abdulhakimfedlu/kddm_lab.git
   cd kddm_lab
   ```

2. **Create and activate virtual environment:**
   ```bash
   # Using uv (recommended)
   uv venv
   .venv\Scripts\activate  # Windows
   
   # Or using standard Python
   python -m venv .venv
   .venv\Scripts\activate  # Windows
   ```

3. **Install dependencies:**
   ```bash
   # Using uv
   uv pip install -e .
   
   # Or using pip
   pip install -e .
   ```

### Running the Notebook

1. **Start JupyterLab:**
   ```bash
   jupyter lab
   ```

2. **Open the notebook** `DWH_OLAP_Assignment_1_Notebook.ipynb`

3. **Run all cells** from top to bottom, or use "Run All Cells" from the menu

The notebook will:
- Load the five CSV files into a DuckDB `raw` schema
- Execute data exploration and quality checks
- Generate the five required exercise queries
- Display results and visualizations

## Data Exploration Process

### 1. Table Inspection

I examined each table using `DESCRIBE` and `SELECT * LIMIT 5` to understand:
- **Grain** – What one row represents in each table
- **Column types** – Data types and nullable columns
- **Sample data** – Representative values and patterns

### 2. Primary Key Validation

I validated primary keys by checking that:
- Row count = distinct value count
- No NULL values exist in the key columns

**Key findings:**
- `customers.customer_id` is unique (surrogate key)
- `orders.order_id` is unique
- `order_items` requires a **composite primary key**: `(order_id, order_item_id)`
- `products.product_id` is unique
- `sellers.seller_id` is unique

### 3. Foreign Key Verification

I used **ANTI JOIN** queries to check for orphan records (child rows with no matching parent):

```sql
SELECT 'orders without customer' AS check_name, count(*) AS n
FROM raw.orders o ANTI JOIN raw.customers c USING (customer_id)
```

**Results:**
- ✓ No orphaned orders (all have valid customers)
- ✓ No orphaned order items (all have valid orders, products, sellers)
- ⚠️ 775 orders have no items (terminal statuses: `unavailable`, `canceled`)

### 4. Cardinality Measurement

I measured min/max children per parent to determine relationship cardinality:

| Relationship | Min | Max | Notation |
|---|---|---|---|
| customers → orders | 1 | 1 | `\|\|--o{` |
| orders → order_items | 0 | 21 | `\|\|--o{` |
| products → order_items | 1 | 527 | `\|\|--o{` |
| sellers → order_items | 1 | 2,033 | `\|\|--o{` |

## The Five Exercises

### Exercise 1: Orders Without Items

**Query:** Find orders with no items, grouped by status.

**Key finding:** 775 orders have no items:
- 603 `unavailable`
- 164 `canceled`
- 5 `created`
- 2 `invoiced`
- 1 `shipped`

### Exercise 2: Order Status Distribution

**Query:** Count and percentage of orders by status.

**Key finding:** Severe class imbalance:
- 97.02% `delivered`
- 1.11% `shipped`
- 0.63% `canceled`
- 0.61% `unavailable`
- All other statuses < 0.32%

### Exercise 3: Missing and Inconsistent Timestamps

**Query:** Count NULL timestamps and logical inconsistencies.

**Key findings:**
- 160 orders missing approval timestamp
- 1,783 orders missing carrier delivery date
- 2,965 orders missing customer delivery date
- 8 orders marked `delivered` but no delivery date
- 23 orders where carrier date > customer date (impossible)

### Exercise 4: Orders Per Month

**Query:** Time series of order counts by month for visualization.

**Key finding:** Dataset spans **September 2016 to October 2018**. First and last months are partial. Peak activity in November 2017 (7,544 orders) and January 2018 (7,269 orders).

### Exercise 5: Product Completeness

**Query:** Count missing values in product attributes.

**Key findings:**
- 32,951 total products
- 610 products missing category, name length, description length, and photo count (same 610 rows)
- Only 2 products missing weight

## Entity-Relationship Diagram

The ERD (`Olist ERD – Crow's Foot Notation (1).pdf`) shows the five tables with:

- All column names and data types
- Primary keys marked (including composite PK on `order_items`)
- Foreign key relationships
- Crow's foot notation indicating cardinality
- Relationship verbs: "places", "contains", "appears in", "fulfils"

**Key relationships:**
- `customers ||--o{ orders` – One customer places zero or many orders
- `orders ||--o{ order_items` – One order contains zero or many items
- `products ||--o{ order_items` – One product appears in zero or many items
- `sellers ||--o{ order_items` – One seller fulfils zero or many items

`order_items` is a **junction table** resolving many-to-many relationships between orders↔products and orders↔sellers.

## Short Note Answers

The `notes.md` file answers three assignment questions:

**a. Two customer identifier columns:**
- `customer_id` (99,441 distinct) – Surrogate key, changes per order
- `customer_unique_id` (96,096 distinct) – Business key, same across orders
- **Use `customer_unique_id` to count customers**

**b. Junction table:**
- `order_items` resolves many-to-many between orders/products and orders/sellers
- Composite PK: `(order_id, order_item_id)`

**c. Stricter data vs. justified circle:**
- **Stricter:** customers→orders shows `||--o{` but data has min=1, max=1 (exactly one)
- **Justified:** orders→order_items shows `||--o{` and 775 orders actually have 0 items

## Main Findings

1. **Data Quality Issues:**
   - 775 orders (0.78%) have no items, mostly due to cancellation or unavailability
   - Significant timestamp missingness (up to 2,965 rows)
   - 23 logical inconsistencies in delivery timestamps

2. **Class Imbalance:**
   - 97% of orders are `delivered`
   - All other statuses combined represent < 3%

3. **Relationship Patterns:**
   - All foreign key relationships are intact (no orphans from child to parent)
   - `customer_id` is not a true unique customer identifier
   - `order_items` is essential for many-to-many resolution

4. **Data Completeness:**
   - Product catalog has 610 incomplete records (1.85%)
   - Most incomplete products lack multiple attributes simultaneously

## Limitations and Observations

- The dataset represents a snapshot and may not include all historical orders
- Temporal coverage is partial at the beginning (Sep 2016) and end (Oct 2018)
- The 775 orders without items suggest incomplete cleanup of failed transactions
- `customer_unique_id` indicates 96,096 unique shoppers made 99,441 orders total, showing some repeat customers (despite `customer_id` appearing 1:1 in the current view)

## Repository

This assignment is submitted through:
**https://github.com/abdulhakimfedlu/kddm_lab**

---

**Submitted by:** Abdulhakim  
**Course:** SWEN466 – Knowledge Discovery and Data Mining  
**Institution:** BITS College, Department of Software Engineering  
**Submission Date:** 26 October 2026
