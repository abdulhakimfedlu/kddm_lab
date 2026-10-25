# KDDM Assignment 1: Data Quality Analysis and ERD Design

## Project Overview

This is my completed Assignment 1 for SWEN466: Knowledge Discovery and Data Mining at BITS College. The assignment required me to explore the Olist e-commerce dataset using DuckDB, complete five SQL exercises analyzing data quality, and construct an entity-relationship diagram (ERD) using crow's foot notation. I validated primary keys, foreign keys, and relationship cardinalities through SQL queries.

## Dataset

I worked with the **Brazilian E-Commerce Public Dataset by Olist**, which includes five CSV files:

- `olist_customers_dataset.csv`
- `olist_orders_dataset.csv`
- `olist_order_items_dataset.csv`
- `olist_products_dataset.csv`
- `olist_sellers_dataset.csv`

All files are in the `data/` directory.

## Tools

- Python 3.14
- Jupyter Notebook
- DuckDB
- pandas
- matplotlib

## Project Files

- `DWH_OLAP_Assignment_1_Notebook.ipynb` – Completed notebook with all five exercises
- `Olist ERD – Crow's Foot Notation (1).pdf` – Entity-relationship diagram
- `notes.md` – Short note answering three assignment questions
- `data/` – Source CSV files
- `pyproject.toml` and `uv.lock` – Project dependencies

## How to Run

1. **Clone the repository:**
   ```bash
   git clone https://github.com/abdulhakimfedlu/kddm_lab.git
   cd kddm_lab
   ```

2. **Install dependencies:**
   ```bash
   # Using uv (recommended)
   uv venv
   .venv\Scripts\activate
   uv pip install -e .
   
   # Or using pip
   python -m venv .venv
   .venv\Scripts\activate
   pip install -e .
   ```

3. **Start Jupyter and open the notebook:**
   ```bash
   jupyter lab
   ```
   
   Open `DWH_OLAP_Assignment_1_Notebook.ipynb` and run all cells.
