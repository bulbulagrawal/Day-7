# Day 18 — Advanced SQL

Today's goal: Move beyond basic SQL queries and learn how professional data analysts use SQL to solve complex business problems.

This is one of the most important days in your SQL learning journey because these concepts frequently appear in Data Analyst interviews, SQL assessments, and real-world reporting tasks.

## 1. Window Functions

A window function performs calculations across a group of related rows without collapsing them into a single row.

Unlike `GROUP BY`, window functions preserve individual records.

### GROUP BY vs Window Functions

Suppose we have an employee table:

| employee\_id | name | department | salary |
| --- | --- | --- | --- |
| 1 | Rahul | IT | 50000 |
| 2 | Priya | IT | 60000 |
| 3 | Amit | HR | 40000 |
| 4 | Neha | HR | 70000 |
| 5 | Karan | IT | 45000 |

Using GROUP BY:

sql

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

Output:

| department | avg\_salary |
| --- | --- |
| IT | 51666.67 |
| HR | 55000 |

Only two rows are returned.

Using a window function:

sql

```sql
SELECT
    employee_id,
    name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS avg_salary
FROM employees;
```

Output:

| name | department | salary | avg\_salary |
| --- | --- | --- | --- |
| Rahul | IT | 50000 | 51666.67 |
| Priya | IT | 60000 | 51666.67 |
| Amit | HR | 40000 | 55000 |
| Neha | HR | 70000 | 55000 |
| Karan | IT | 45000 | 51666.67 |

Remember:

- `GROUP BY` reduces rows.
- `OVER()` performs calculations while retaining rows.
- `PARTITION BY` divides data into groups.
- `ORDER BY` inside `OVER()` determines calculation order.

General syntax:

sql

```sql
SELECT
    column_name,
    WINDOW_FUNCTION() OVER (
        PARTITION BY column_name
        ORDER BY column_name
    )
FROM table_name;
```

## 2. ROW\_NUMBER()

Assigns a unique sequential number to each row within a window.

Example: Rank employees by salary within each department.

sql

```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS row_num
FROM employees;
```

Output:

| name | department | salary | row\_num |
| --- | --- | --- | --- |
| Priya | IT | 60000 | 1 |
| Rahul | IT | 50000 | 2 |
| Karan | IT | 45000 | 3 |
| Neha | HR | 70000 | 1 |
| Amit | HR | 40000 | 2 |

Business use cases:

- Find the latest order for each customer.
- Find the first transaction.
- Remove duplicate records.
- Select the top employee in each department.

Example: Find the latest order per customer.

sql

```sql
WITH ranked_orders AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC
        ) AS rn
    FROM orders
)

SELECT *
FROM ranked_orders
WHERE rn = 1;
```

## 3. RANK()

Assigns ranking based on values.

If two rows have the same value, they receive the same rank. The next rank is skipped.

Example:

| Employee | Salary | RANK |
| --- | --- | --- |
| Priya | 60000 | 1 |
| Rahul | 50000 | 2 |
| Karan | 50000 | 2 |
| Amit | 40000 | 4 |

sql

```sql
SELECT
    name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

## 4. DENSE\_RANK()

Similar to RANK, but it does not skip ranking numbers.

| Employee | Salary | DENSE\_RANK |
| --- | --- | --- |
| Priya | 60000 | 1 |
| Rahul | 50000 | 2 |
| Karan | 50000 | 2 |
| Amit | 40000 | 3 |

sql

```sql
SELECT
    name,
    salary,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

### ROW\_NUMBER vs RANK vs DENSE\_RANK

| Function | Duplicate values | Ranking |
| --- | --- | --- |
| ROW\_NUMBER | Different numbers | 1, 2, 3, 4 |
| RANK | Same rank | 1, 2, 2, 4 |
| DENSE\_RANK | Same rank | 1, 2, 2, 3 |

Interview favoriteQuestion: Find the second-highest salary.

sql

```sql
WITH salary_rank AS (
    SELECT
        name,
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
)

SELECT *
FROM salary_rank
WHERE rnk = 2;
```

This returns all employees earning the second-highest distinct salary.

## 5. LAG()

`LAG()` accesses a value from a previous row without joining the table to itself.

Example: Compare each month's sales with the previous month.

| Month | Sales |
| --- | --- |
| January | 10000 |
| February | 15000 |
| March | 12000 |
| April | 20000 |

sql

```sql
SELECT
    month,
    sales,
    LAG(sales) OVER (
        ORDER BY month
    ) AS previous_month_sales
FROM monthly_sales;
```

Output:

| Month | Sales | Previous sales |
| --- | --- | --- |
| January | 10000 | NULL |
| February | 15000 | 10000 |
| March | 12000 | 15000 |
| April | 20000 | 12000 |

## 6. LEAD()

`LEAD()` accesses a value from the next row.

sql

```sql
SELECT
    month,
    sales,
    LEAD(sales) OVER (
        ORDER BY month
    ) AS next_month_sales
FROM monthly_sales;
```

Output:

| Month | Sales | Next sales |
| --- | --- | --- |
| January | 10000 | 15000 |
| February | 15000 | 12000 |
| March | 12000 | 20000 |
| April | 20000 | NULL |

Remember:

- `LAG()` = Previous row.
- `LEAD()` = Next row.

Common business applications:

- Month-over-month comparisons.
- Customer purchase behavior.
- Sales trends.
- Identifying gaps between transactions.

## 7. Running Totals

A running total calculates the cumulative sum of values over time.

Example:

| Month | Sales |
| --- | --- |
| January | 10000 |
| February | 15000 |
| March | 12000 |
| April | 20000 |

Expected running total:

| Month | Sales | Running Total |
| --- | --- | --- |
| January | 10000 | 10000 |
| February | 15000 | 25000 |
| March | 12000 | 37000 |
| April | 20000 | 57000 |

SQL:

sql

```sql
SELECT
    month,
    sales,
    SUM(sales) OVER (
        ORDER BY month
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS running_total
FROM monthly_sales;
```

Explanation:

- `SUM(sales)` calculates the sum.
- `ORDER BY month` defines the sequence.
- `UNBOUNDED PRECEDING` starts from the first row.
- `CURRENT ROW` calculates up to the current row.

Business use cases:

- Cumulative revenue.
- Year-to-date sales.
- Cumulative customer registrations.
- Running expense tracking.

## 8. Moving Calculations

A moving calculation calculates values over a sliding window of rows.

For example, a 3-month moving average calculates the average sales of the current month and previous two months.

sql

```sql
SELECT
    month,
    sales,
    AVG(sales) OVER (
        ORDER BY month
        ROWS BETWEEN 2 PRECEDING
        AND CURRENT ROW
    ) AS moving_avg
FROM monthly_sales;
```

Example:

| Month | Sales | 3-month moving average |
| --- | --- | --- |
| January | 10000 | 10000 |
| February | 15000 | 12500 |
| March | 12000 | 12333.33 |
| April | 20000 | 15666.67 |

Why use moving averages?

- Identify sales trends.
- Reduce short-term fluctuations.
- Analyze business growth.
- Understand seasonal patterns.

Other useful window frames:

sql

```sql
-- Previous row
ROWS BETWEEN 1 PRECEDING AND CURRENT ROW

-- Previous 2 rows and current row
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW

-- All previous rows and current row
ROWS BETWEEN UNBOUNDED PRECEDING
AND CURRENT ROW
```

## 9. Month-over-Month (MoM) Analysis

MoM analysis compares a business metric with the previous month.

Example:

| Month | Revenue |
| --- | --- |
| January | 100000 |
| February | 120000 |
| March | 150000 |
| April | 135000 |

Business questions:

- How much did revenue grow?
- Did sales decrease?
- What is the monthly growth percentage?

### SQL solution

sql

```sql
WITH monthly_revenue AS (
    SELECT
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
),

revenue_comparison AS (
    SELECT
        month,
        revenue,
        LAG(revenue) OVER (
            ORDER BY month
        ) AS previous_revenue
    FROM monthly_revenue
)

SELECT
    month,
    revenue,
    previous_revenue,
    revenue - previous_revenue AS growth,
    ROUND(
        (revenue - previous_revenue)
        / NULLIF(previous_revenue, 0) * 100,
        2
    ) AS mom_growth_percentage
FROM revenue_comparison;
```

### Expected output

| Month | Revenue | Previous | Growth % |
| --- | --- | --- | --- |
| January | 100000 | NULL | NULL |
| February | 120000 | 100000 | 20% |
| March | 150000 | 120000 | 25% |
| April | 135000 | 150000 | -10% |

Formula:

$$
MoM\ Growth =
\frac{Current-Previous}{Previous}\times100
$$

Important for data analystsMoM analysis is commonly used in:

- Monthly revenue reports.
- Customer acquisition analysis.
- Monthly sales performance.
- Expense tracking.
- Business growth dashboards.

Note: This example uses MySQL date formatting. For reliable monthly analysis, use a proper month-start date and ensure months with no orders are included if the business requires a complete monthly series.

## 10. Top-N Analysis

Top-N means finding the top N records based on a particular metric.

Examples:

- Top 5 customers by revenue.
- Top 10 products by sales.
- Top 3 employees by salary.
- Top 5 cities by orders.

### Example 1: Top 3 customers by spending

sql

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 3;
```

### Example 2: Top 3 employees in each department

sql

```sql
WITH employee_ranking AS (
    SELECT
        employee_id,
        employee_name,
        department_id,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC, employee_id
        ) AS rn
    FROM employees
)

SELECT *
FROM employee_ranking
WHERE rn <= 3;
```

Important distinction:

- `LIMIT 3` gives three rows overall.
- `ROW_NUMBER() PARTITION BY department_id` gives up to three employees per department.
- Use `RANK()` or `DENSE_RANK()` instead when ties should share a rank.

## 11. Customer Segmentation

Customer segmentation means dividing customers into groups based on their purchasing behavior.

One common approach is based on total customer spending.

Example:

| Customer | Total Spending |
| --- | --- |
| Rahul | 100000 |
| Priya | 50000 |
| Amit | 15000 |
| Neha | 3000 |

We can create three segments:

- High Value: Spending above 50000.
- Medium Value: Spending between 10000 and 50000.
- Low Value: Spending below 10000.

### SQL using CASE WHEN

sql

```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)

SELECT
    customer_id,
    total_spending,
    CASE
        WHEN total_spending > 50000
            THEN 'High Value'

        WHEN total_spending >= 10000
            THEN 'Medium Value'

        ELSE 'Low Value'
    END AS customer_segment
FROM customer_spending;
```

### Expected output

| Customer | Spending | Segment |
| --- | --- | --- |
| Rahul | 100000 | High Value |
| Priya | 50000 | Medium Value |
| Amit | 15000 | Medium Value |
| Neha | 3000 | Low Value |

### Advanced concept: RFM Segmentation

RFM is a popular customer analysis technique.

| Metric | Meaning |
| --- | --- |
| Recency | How recently did the customer purchase? |
| Frequency | How often does the customer purchase? |
| Monetary | How much money does the customer spend? |

RFM customer analysis

# R

Recency

Last purchase date

# F

Frequency

Number of purchases

# M

Monetary

Total spending

Example segments:

- Champions: Recent, frequent, high spending.
- Loyal customers: Frequent purchases.
- At-risk customers: Haven't purchased recently.
- New customers: Recently made their first purchase.

## 12. SQL Optimization Basics

SQL optimization means improving query performance so queries execute efficiently.

Imagine a database containing 10 million customer records.

A poorly written query can take several seconds or minutes, whereas an optimized query may execute much faster.

### Important optimization techniques

1. Avoid SELECT \*

Instead of:

sql

```sql
SELECT *
FROM customers;
```

Prefer:

sql

```sql
SELECT
    customer_id,
    customer_name,
    email
FROM customers;
```

Why? Retrieve only the columns you need.

2. Use WHERE filters

sql

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01';
```

Filter unnecessary records early.

3. Avoid unnecessary DISTINCT

sql

```sql
SELECT DISTINCT customer_id
FROM orders;
```

Use DISTINCT only when duplicate elimination is required.

4. Avoid unnecessary subqueries

Sometimes a JOIN or CTE can make the query easier to understand and optimize.

5. Use indexes appropriately

Indexes can significantly improve searches and joins on suitable columns.

6. Analyze execution plans

sql

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 101;
```

`EXPLAIN` helps you understand how the database plans to execute a query.

In MySQL, `EXPLAIN ANALYZE` can also provide actual execution information.

## 13. Index Fundamentals

An index is a database structure that helps the database find records faster.

Think of a book.

Without an index, you might need to read every page to find a particular topic.

With an index, you can quickly locate the relevant page.

A database index works on a similar principle.

Optimizing Databases: A Comprehensive Guide with Real-World Examples | by Purnendu Kar | Nerd For Tech | Medium

### Creating an index

sql

```sql
CREATE INDEX idx_customer_id
ON orders(customer_id);
```

Now queries filtering by customer ID may become faster.

sql

```sql
SELECT *
FROM orders
WHERE customer_id = 101;
```

### Types of indexes

| Index type | Explanation |
| --- | --- |
| Primary key index | Supports primary key uniqueness |
| Unique index | Prevents duplicate indexed values |
| Composite index | Indexes multiple columns |
| Clustered index | Organizes table data according to an index |
| Non-clustered index | Stores index entries separately from table rows |

Note: Exact index implementations vary by database engine. For example, InnoDB uses a clustered primary-key index.

### Composite index example

sql

```sql
CREATE INDEX idx_customer_date
ON orders(customer_id, order_date);
```

This can help queries such as:

sql

```sql
SELECT *
FROM orders
WHERE customer_id = 101
AND order_date >= '2026-01-01';
```

Important index rules:

- Index columns frequently used in filtering and joins.
- Composite index column order matters.
- Too many indexes can slow down INSERT, UPDATE, and DELETE operations.
- Indexes consume storage.
- Not every query benefits from an index.
- Always examine the execution plan.

# 14. Day 18 — Real-World Business Analysis Challenges

Now let's combine everything you've learned.

Assume these tables:

CUSTOMERS

| customer\_id | customer\_name | city |
| --- | --- | --- |
| 1 | Rahul | Indore |
| 2 | Priya | Mumbai |
| 3 | Amit | Delhi |
| 4 | Neha | Pune |

ORDERS

| order\_id | customer\_id | order\_date | amount |
| --- | --- | --- | --- |
| 101 | 1 | 2026-01-05 | 5000 |
| 102 | 2 | 2026-01-10 | 8000 |
| 103 | 1 | 2026-02-05 | 3000 |
| 104 | 3 | 2026-02-15 | 10000 |
| 105 | 2 | 2026-03-10 | 12000 |
| 106 | 4 | 2026-03-15 | 2000 |

## Practice questions

Try solving these independently before checking solutions.

| No. | Business problem | Concept |
| --- | --- | --- |
| 1 | Find the second-highest order amount. | DENSE\_RANK |
| 2 | Find the highest-spending customer. | GROUP BY |
| 3 | Calculate cumulative monthly revenue. | Running total |
| 4 | Calculate previous month's revenue. | LAG |
| 5 | Calculate next month's revenue. | LEAD |
| 6 | Calculate monthly revenue growth %. | MoM |
| 7 | Find the top 2 customers by spending. | Top-N |
| 8 | Rank customers by total spending. | RANK |
| 9 | Calculate a 3-month moving average. | Window functions |
| 10 | Segment customers into high, medium and low value. | CASE WHEN |

## 15. Interview-Level SQL Practice

Here are five additional challenges.

Challenge 1: Find the latest order for every customer.

Challenge 2: Find customers who placed orders in consecutive months.

Challenge 3: Find customers whose current month's spending is higher than the previous month's spending.

Challenge 4: Find the top 3 products by revenue within each category.

Challenge 5: Identify customers who have not purchased anything in the last 90 days.

These problems test whether you can combine JOINs, CTEs, subqueries, and window functions.

## 17. Day 18 Completion Checklist

- [ ]Understand window functions.
- [ ]Practice ROW\_NUMBER, RANK and DENSE\_RANK.
- [ ]Understand LAG and LEAD.
- [ ]Calculate running totals.
- [ ]Calculate moving averages.
- [ ]Perform month-over-month analysis.
- [ ]Solve Top-N problems.
- [ ]Perform customer segmentation.
- [ ]Understand SQL optimization.
- [ ]Understand database indexes.
- [ ]Solve at least 10 business analysis problems.

## Your SQL learning milestone

# Advanced SQL concepts covered\!

You've now reached an important milestone in your SQL learning journey.

| Learning phase | Concepts |
| --- | --- |
| SQL Fundamentals | SELECT, WHERE, ORDER BY |
| Aggregations | GROUP BY, HAVING |
| Intermediate SQL | JOINs, subqueries, CTEs |
| Advanced SQL | Window functions, MoM, Top-N |
| Performance | Optimization, indexes |

My recommendation: Don't just memorize window function syntax. Practice solving business problems using these functions. That is what makes the difference in SQL interviews.

Next step: Try solving the 10 business analysis questions without looking at the answers. Focus especially on LAG, DENSE\_RANK, running totals, and customer segmentation.
