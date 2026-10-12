# Grocery Store Management — SQL Analytics Project

This project models day-to-day grocery-store sales data in MySQL. It provides a relational database design, CSV source data, date-cleanup steps, and ready-to-run SQL queries for exploring customers, products, orders, suppliers, and employee performance.

## Contents

| File | Purpose |
| --- | --- |
| `Grocery_Store_Mangement.sql` | Creates the `SalesDB` database and tables, converts imported text dates to `DATE`, and contains analysis queries. |
| `Categories.csv` | Product categories. |
| `Suppliers.csv` | Supplier names and addresses. |
| `Store_Employees.csv` | Employee names and hire dates. |
| `Customers.csv` | Customer names and addresses. |
| `Products.csv` | Products, supplier/category references, and list prices. |
| `Orders.csv` | Customer orders, processing employee, and order date. |
| `OrderDetails.csv` | Individual order lines, quantities, unit prices, and line totals. |

## Database design

The database is named `SalesDB` and has seven related tables:

```text
supplier (sup_id) ──< products (prod_id) >── categories (cat_id)
                           |
                           v
customers (cust_id) ──< orders (ord_id) ──< order_details (ord_detID) >── products
                         ^
                         |
                    employees (emp_id)
```

| Table | Primary key | Description |
| --- | --- | --- |
| `supplier` | `sup_id` | Supplier master data. |
| `categories` | `cat_id` | Product categories. |
| `employees` | `emp_id` | Employees who process orders. |
| `customers` | `cust_id` | Customer master data. |
| `products` | `prod_id` | Products with supplier, category, and price. |
| `orders` | `ord_id` | Order header with customer, employee, and order date. |
| `order_details` | `ord_detID` | Order line items with quantity and price totals. |

Foreign keys preserve these relationships. They use `ON UPDATE CASCADE` and `ON DELETE CASCADE`, so changing or deleting a parent record also updates or removes dependent records.

## Requirements

- MySQL 8.0+ (or a compatible MySQL version)
- A MySQL client such as MySQL Workbench

## Setup and data import

1. Run the **database setup** and **table creation** sections of `Grocery_Store_Mangement.sql`.
2. Import the CSV files into their matching tables in this order, which satisfies foreign-key dependencies:

   1. `Suppliers.csv` → `supplier`
   2. `Categories.csv` → `categories`
   3. `Store_Employees.csv` → `employees`
   4. `Customers.csv` → `customers`
   5. `Products.csv` → `products`
   6. `Orders.csv` → `orders`
   7. `OrderDetails.csv` → `order_details`

   In MySQL Workbench, use **Table Data Import Wizard** and map the CSV headers to the corresponding database columns. For `order_details`, either import `OrderDetailID` into `ord_detID` or omit it and let `AUTO_INCREMENT` generate the IDs.

3. Run the **date conversion after CSV import** section. It temporarily stores imported date text, converts it to `DATE`, then replaces the original text columns.
4. Run any query in the **SQL analysis section** to explore the data.

> **Date-format note:** the script converts employee hire dates using the `MM/DD/YYYY` pattern and order dates using `MM/DD/YYYY` or `DD-MM-YY`. Check the actual CSV format before conversion—`Store_Employees.csv` currently shows hyphenated dates—then adjust the `STR_TO_DATE` format if needed (for example, `'%d-%m-%y'`).

## Analysis included

The SQL script answers practical reporting questions in six areas:

- **Customer insights:** number of ordering customers, orders per customer, purchase totals/averages, and top five customers by spend.
- **Product performance:** products per category, average category price, units sold, revenue by product, and category/supplier/product revenue.
- **Sales and order trends:** total orders, order value, busiest dates, monthly revenue and order volume, and weekday versus weekend activity.
- **Supplier contribution:** supplier count, product coverage, average prices, and sales revenue attributable to each supplier.
- **Employee performance:** employees who processed orders, orders handled, sales handled, and average order value.
- **Order-detail exploration:** quantity/price relationships, average quantities per product, and observed unit-price variation.

## Important implementation details

- Monetary values use `DECIMAL(10,2)` in the schema.
- `order_details.total_price` is stored in the imported data and is used in the analysis queries; it is not calculated automatically by the database.
- Initial `hire_date` and `order_date` columns are `VARCHAR(255)` to permit CSV import. The conversion section changes both to `DATE` afterward.
- Run the date conversion section once on a freshly imported schema. Running its `ADD COLUMN` statements again without resetting the tables will cause duplicate-column errors.

## Example query

```sql
SELECT p.prod_name, SUM(od.total_price) AS total_revenue
FROM products AS p
JOIN order_details AS od ON od.prod_id = p.prod_id
GROUP BY p.prod_id, p.prod_name
ORDER BY total_revenue DESC;
```

This returns products ranked by their total sales revenue.
