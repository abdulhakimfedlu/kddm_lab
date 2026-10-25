# Data Quality Notes – Olist Dataset

## a. Two customer identifier columns

The `customers` table contains `customer_id` (99,441 distinct values, equal to row count) and `customer_unique_id` (96,096 distinct values). Since `customer_id` is unique per row but `customer_unique_id` has fewer distinct values, the dataset represents each order as a separate customer record. Use `customer_unique_id` to count unique customers (96,096), and `customer_id` to join orders to customers.

## b. Junction table

`order_items` is the junction table with composite primary key `(order_id, order_item_id)`. It resolves many-to-many relationships between orders and products (one order can have many products; one product appears in many orders) and between orders and sellers (one order can involve multiple sellers; one seller fulfills many orders).

## c. Data stricter than diagram vs. circle justified

**Stricter:** The diagram shows `customers ||--o{ orders` (zero or many), but every `customer_id` has exactly one order (`min=1, max=1`), making the data stricter than the diagram allows. **Circle justified:** The diagram shows `orders ||--o{ order_items` (zero or many), and 775 orders actually have zero items, confirming the circle (optional relationship) is correct.
