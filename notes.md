# Data Quality Notes – Olist Dataset

## a. The two customer identifier columns

The `customers` table has two columns that look like identifiers:
`customer_id` (99 441 distinct values, equals the row count) and
`customer_unique_id` (only 96 096 distinct values).
`customer_id` is a per-order surrogate: the same real customer gets a fresh
`customer_id` every time they place an order, so a single person can appear in
multiple rows.
`customer_unique_id` is the true business identifier that stays the same across
orders; **use `customer_unique_id` to count customers** (96 096 unique shoppers)
and `customer_id` only to join `orders` to `customers`.

## b. The junction table

`order_items` is the junction table.
It resolves the many-to-many relationship between `orders` and `products`
(one order can contain many products, and one product can appear in many orders)
and the many-to-many relationship between `orders` and `sellers`
(one order can involve several sellers, and one seller fulfils many orders).
Its composite primary key is `(order_id, order_item_id)`.

## c. Stricter in data vs. justified circle in diagram

**Stricter in the data:** the diagram shows `customers ||--o{ orders`
(one customer, zero-or-more orders), because in principle a customer record could
exist before any order is placed.
The data, however, shows every customer has **exactly one** order
(`min = 1, max = 1` orders per `customer_id`), so the actual data
is stricter than the crow's-foot "zero or many" symbol on the diagram.

**Circle justified by data:** `orders o{--|| order_items`
(one order, zero-or-many items) is justified because **775 orders** have no
matching rows in `order_items` at all (all are in terminal statuses such as
`unavailable`, `canceled`, or `created`), confirming that the "zero" side of the
relationship genuinely occurs in the data.

# Entity Relationship Diagram (Text Format)

```
┌─────────────────────────────────┐
│         CUSTOMERS               │
├─────────────────────────────────┤
│ • customer_id (PK)              │
│   customer_unique_id            │
│   customer_zip_code_prefix      │
│   customer_city                 │
│   customer_state                │
└──────────────┬──────────────────┘
               │ places
               │ (1 customer → 0..* orders)
               ↓
┌─────────────────────────────────┐
│           ORDERS                │
├─────────────────────────────────┤
│ • order_id (PK)                 │
│   customer_id (FK)              │
│   order_status                  │
│   order_purchase_timestamp      │
│   order_approved_at             │
│   order_delivered_carrier_date  │
│   order_delivered_customer_date │
│   order_estimated_delivery_date │
└──────────────┬──────────────────┘
               │ contains
               │ (1 order → 0..* items)
               ↓
┌──────────────────────────────────┐       ┌────────────────────────┐
│        ORDER_ITEMS               │       │      PRODUCTS          │
├──────────────────────────────────┤       ├────────────────────────┤
│ • order_id (PK, FK)             │←──────│ • product_id (PK)      │
│ • order_item_id (PK)            │       │   product_category_name│
│   product_id (FK)               │       │   product_name_lenght  │
│   seller_id (FK)                │       │   product_description..│
│   shipping_limit_date           │       │   product_photos_qty   │
│   price                         │       │   product_weight_g     │
│   freight_value                 │       │   ... (dimensions)     │
└───────────────┬──────────────────┘       └────────────────────────┘
                │                            (1 product → 1..* items)
                │ fulfils
                │ (1 seller → 1..* items)
                ↓
      ┌─────────────────────┐
      │      SELLERS        │
      ├─────────────────────┤
      │ • seller_id (PK)    │
      │   seller_zip_code..│
      │   seller_city       │
      │   seller_state      │
      └─────────────────────┘
```

**Crow's Foot Notation Legend:**
- `||` = exactly one (mandatory)
- `o{` = zero or many (optional many)
- PK = Primary Key
- FK = Foreign Key
