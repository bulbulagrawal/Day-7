# Day 16 — SQL JOINs

Today's goal: Understand how to combine data from multiple tables using JOINs.

Imagine an e-commerce database where customer information, orders, order items, and products are stored separately.

## 1. Understanding table relationships

CUSTOMERS

customer\_id (PK)

customer\_name · email

One-to-Manycustomer\_id (FK)

ORDERS

order\_id (PK) · customer\_id (FK)

order\_date · total\_amount

One-to-Manyorder\_id (FK)

ORDER\_ITEMS

order\_item\_id (PK)

order\_id (FK) · product\_id (FK) · quantity

Many-to-Oneproduct\_id (FK)

PRODUCTS

product\_id (PK)

product\_name · price

PK = Primary Key

FK = Foreign Key

## 2. Types of SQL JOINs

### A. INNER JOIN

Returns only matching records from both tables.

A Complete Guide to Different Types of Joins in SQL

sql

```sql
SELECT
    customers.customer_name,
    orders.order_id
FROM customers
INNER JOIN orders
ON customers.customer_id = orders.customer_id;
```

Meaning: Show customers who have placed orders.

### B. LEFT JOIN

Returns all records from the LEFT table and matching records from the RIGHT table.

Joining movie ratings with customer data | SQL

sql

```sql
SELECT
    customers.customer_name,
    orders.order_id
FROM customers
LEFT JOIN orders
ON customers.customer_id = orders.customer_id;
```

Meaning: Show all customers, including those who have never ordered.

Customers without orders will have `NULL` in the order columns.

### C. RIGHT JOIN

Returns all records from the RIGHT table and matching records from the LEFT table.

A beginner’s guide to 7 types of SQL JOINs | TablePlus

sql

```sql
SELECT
    customers.customer_name,
    orders.order_id
FROM customers
RIGHT JOIN orders
ON customers.customer_id = orders.customer_id;
```

Meaning: Show all orders, including orders without a matching customer.

### D. FULL OUTER JOIN

Returns all matching and non-matching records from both tables.

Different Types of SQL JOINs that You Must Know - StrataScratch

sql

```sql
SELECT
    customers.customer_name,
    orders.order_id
FROM customers
FULL OUTER JOIN orders
ON customers.customer_id = orders.customer_id;
```

Meaning: Show every customer and every order, whether they match or not.

Note: MySQL does not directly support `FULL OUTER JOIN`. You can simulate it using `LEFT JOIN UNION RIGHT JOIN`.

### E. SELF JOIN

A table joins with itself.

Example: Employees and their managers are stored in the same table.

sql

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.employee_id;
```

Meaning: Find each employee's manager.

## 3. JOIN comparison

| JOIN | What it returns |
| --- | --- |
| INNER | Matching records only |
| LEFT | All left + matching right |
| RIGHT | All right + matching left |
| FULL | Everything from both |
| SELF | Table joined with itself |

## 4. Database relationships

### One-to-One (1:1)

One person has one passport.

ER DIAGRAM & ER MODELING IN DBMS | PPTX

Example:

- PERSON → PASSPORT

### One-to-Many (1:N)

One customer can place multiple orders.

Denormalization | System Design

Example:

- CUSTOMER → ORDERS

### Many-to-Many (M:N)

One order can contain multiple products, and one product can belong to multiple orders.

Relational Database Design

Example:

- ORDERS ↔ PRODUCTS

We use `ORDER_ITEMS` as a junction (bridge) table to implement this relationship.

## 5. Practical SQL JOIN exercise

Assume these tables:

CUSTOMERS

| customer\_id | name |
| --- | --- |
| 1 | Rahul |
| 2 | Priya |
| 3 | Amit |
| 4 | Neha |

ORDERS

| order\_id | customer\_id |
| --- | --- |
| 101 | 1 |
| 102 | 1 |
| 103 | 2 |
| 104 | 4 |


Day 16 takeaway: JOINs connect related tables. Relationships explain how those tables are connected. Both are essential for writing real-world SQL queries.
