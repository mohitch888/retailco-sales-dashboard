# Data Dictionary: `retail_store_sales.csv`

12,575 rows, 11 columns, 2022-01-01 to 2025-01-18. `Transaction ID` is unique (no duplicates).

| Column | Type | Description | Missing (raw) |
|---|---|---|---|
| Transaction ID | Text | Unique transaction identifier, e.g. `TXN_6867343` | 0 |
| Customer ID | Text | Customer identifier (25 distinct values, `CUST_01`...) | 0 |
| Category | Text | Product category (8 values) | 0 |
| Item | Text | Product code, e.g. `Item_25_FUR` (200 distinct) | 1,213 |
| Price Per Unit | Decimal | Unit price (range about 5 to 41) | 609 |
| Quantity | Integer | Units purchased (1 to 10) | 604 |
| Total Spent | Decimal | Transaction value; always equals Price Per Unit x Quantity | 604 |
| Payment Method | Text | Cash, Credit Card, Digital Wallet | 0 |
| Location | Text | Online or In-store (shown as "Channel" in the report) | 0 |
| Transaction Date | Date | Date of purchase (YYYY-MM-DD) | 0 |
| Discount Applied | Boolean | True / False; blank when unknown | 4,199 |

## Categories

Butchers, Beverages, Computers and electric accessories, Electric household essentials, Food, Furniture, Milk Products, Patisserie

## Relationships verified

- Each Item has exactly one Price Per Unit and belongs to exactly one Category.
- Each Category + Price Per Unit pair maps to exactly one Item, which makes Item recovery safe.
- Price Per Unit x Quantity = Total Spent on 100% of complete rows.
