# Day 17 — SQL Subqueries, UNION & CTEs

Today's goal: Learn how to combine multiple query results, write queries inside queries, and simplify complex SQL using CTEs.

These concepts are especially important for SQL interviews, data analyst roles, and real-world business analysis.

## 1. UNION

`UNION` combines results from two or more SELECT queries and removes duplicate rows.

Example:

Suppose we have two tables:

ONLINE\_CUSTOMERS

| customer\_id | name |
| --- | --- |
| 1 | Rahul |
| 2 | Priya |

STORE\_CUSTOMERS

| customer\_id | name |
| --- | --- |
| 2 | Priya |
| 3 | Amit |

sql

```sql
SELECT customer_id, name
FROM online_customers

UNION

SELECT customer_id, name
FROM store_customers;
```

Output:

| customer\_id | name |
| --- | --- |
| 1 | Rahul |
| 2 | Priya |
| 3 | Amit |

Notice that Priya appears only once.

Rules of UNION:

- Both queries must return the same number of columns.
- Corresponding columns must have compatible data types.
- Column names in the result come from the first SELECT.
- Duplicate rows are removed.

## 2. UNION ALL

`UNION ALL` combines results but keeps duplicates.

sql

```sql
SELECT customer_id, name
FROM online_customers

UNION ALL

SELECT customer_id, name
FROM store_customers;
```

Output:

| customer\_id | name |
| --- | --- |
| 1 | Rahul |
| 2 | Priya |
| 2 | Priya |
| 3 | Amit |

### UNION vs UNION ALL

| UNION | UNION ALL |
| --- | --- |
| Removes duplicates | Keeps duplicates |
| Usually slower | Usually faster |
| Performs duplicate elimination | No duplicate elimination |
| Useful for unique lists | Useful for combining all records |

Interview tipUse `UNION ALL` when duplicates are acceptable or meaningful. Use `UNION` when you need distinct combined results.

## 3. Subqueries

A subquery is a query written inside another SQL query.

In simple words:

First, one query gets the result. Then the outer query uses that result.

Example: Find employees earning more than the average salary.

EMPLOYEES

| employee\_id | name | salary |
| --- | --- | --- |
| 1 | Rahul | 30000 |
| 2 | Priya | 50000 |
| 3 | Amit | 40000 |
| 4 | Neha | 60000 |

sql

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

How it works:

Step 1: Inner query calculates average salary.

sql

```sql
SELECT AVG(salary)
FROM employees;
```

Result:

text

```text
45000
```

Step 2: Outer query finds employees earning more than 45000.

Output:

| employee\_id | name | salary |
| --- | --- | --- |
| 2 | Priya | 50000 |
| 4 | Neha | 60000 |

### Types of subqueries

- Scalar subquery: Returns one value.
- Single-row subquery: Returns one row.
- Multiple-row subquery: Returns multiple rows.
- Correlated subquery: Depends on the outer query.

### Common operators

| Operator | Purpose |
| --- | --- |
| IN | Matches values from a list |
| NOT IN | Excludes values from a list |
| EXISTS | Checks whether rows exist |
| ANY | Compares with at least one value |
| ALL | Compares with every value |

Example using IN:

sql

```sql
SELECT *
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE department_name = 'IT'
);
```

## 4. Correlated Subquery

A correlated subquery is a subquery that depends on the outer query.

Unlike a normal subquery, it references a column from the outer query.

Business problem: Find employees earning more than the average salary of their own department.

sql

```sql
SELECT
    e.employee_id,
    e.employee_name,
    e.department_id,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

How it works:

1. Outer query selects an employee.
2. Inner query calculates the average salary of that employee's department.
3. SQL compares the employee's salary with that average.
4. The process is evaluated for each outer employee row.

Example:

| Employee | Department | Salary |
| --- | --- | --- |
| Rahul | IT | 50000 |
| Priya | IT | 40000 |
| Amit | HR | 60000 |
| Neha | HR | 30000 |

Department averages:

- IT = 45000
- HR = 45000

Output:

| Employee | Department | Salary |
| --- | --- | --- |
| Rahul | IT | 50000 |
| Amit | HR | 60000 |

Important: Correlated subqueries are conceptually evaluated for each outer row, although the database optimizer may execute them differently.

## 5. CTE (Common Table Expression)

A CTE is a temporary named result set that exists for the duration of a single SQL statement.

It makes complex SQL queries easier to read and maintain.

Syntax:

sql

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT *
FROM cte_name;
```

Example: Find employees earning more than 40000.

sql

```sql
WITH high_salary AS (
    SELECT *
    FROM employees
    WHERE salary > 40000
)

SELECT *
FROM high_salary;
```

Here:

- `WITH` creates the CTE.
- `high_salary` is its name.
- The inner SELECT creates its result.
- The outer SELECT uses that result.

What Is a CTE? | LearnSQL.com

### Subquery vs CTE

| Subquery | CTE |
| --- | --- |
| Query inside another query | Named query using WITH |
| Can become difficult to read | Often easier to read |
| Can be nested | Can be referenced multiple times within a statement |
| Useful for simple logic | Useful for complex logic |

Note: A CTE is not necessarily stored as a physical temporary table. Its execution and materialization depend on the database engine.

## 6. Multiple CTEs

You can create multiple CTEs in a single SQL statement.

Business problem: Find departments whose average salary is above 45000.

sql

```sql
WITH department_salary AS (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),

high_salary_departments AS (
    SELECT *
    FROM department_salary
    WHERE avg_salary > 45000
)

SELECT *
FROM high_salary_departments;
```

How it works:

CTE 1: department\_salary

Calculate average salary for each department

CTE 2: high\_salary\_departments

Filter departments with average salary above 45000

Final SELECT

Display the final result

# 7. Business Analysis Problems

Now let's apply these concepts to real-world business scenarios.

Assume an e-commerce database with these tables:

CUSTOMERS

| customer\_id | customer\_name | city |
| --- | --- | --- |
| 1 | Rahul | Indore |
| 2 | Priya | Mumbai |
| 3 | Amit | Delhi |

ORDERS

| order\_id | customer\_id | amount |
| --- | --- | --- |
| 101 | 1 | 5000 |
| 102 | 1 | 3000 |
| 103 | 2 | 7000 |
| 104 | 3 | 2000 |

PRODUCTS

| product\_id | product\_name | price |
| --- | --- | --- |
| 1 | Laptop | 50000 |
| 2 | Mouse | 500 |
| 3 | Keyboard | 1500 |

### Problem 1: Find customers who spent more than the average order amount.

sql

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE c.customer_id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > (
        SELECT AVG(amount)
        FROM orders
    )
);
```

### Problem 2: Find the highest-spending customer.

sql

```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
)

SELECT
    c.customer_name,
    cs.total_spent
FROM customer_spending cs
JOIN customers c
ON cs.customer_id = c.customer_id
ORDER BY cs.total_spent DESC
LIMIT 1;
```

### Problem 3: Find customers who have never placed an order.

sql

```sql
SELECT *
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id
    FROM orders
);
```

In production, `NOT EXISTS` is often safer than `NOT IN` when the subquery might contain NULL values.

### Problem 4: Find customers whose spending is above the average customer spending.

sql

```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
)

SELECT *
FROM customer_spending
WHERE total_spent > (
    SELECT AVG(total_spent)
    FROM customer_spending
);
```

### Problem 5: Find the second-highest order amount.

sql

```sql
SELECT MAX(amount) AS second_highest
FROM orders
WHERE amount < (
    SELECT MAX(amount)
    FROM orders
);
```

## 8. Quick Revision

| Concept | Remember |
| --- | --- |
| UNION | Combines results, removes duplicates |
| UNION ALL | Combines results, keeps duplicates |
| Subquery | Query inside another query |
| Correlated subquery | Depends on outer query |
| CTE | Named temporary result set |
| Multiple CTEs | Multiple named result sets in one statement |

## 9. Day 17 — Practice Test

Test yourself before moving to Day 18.

SQL Knowledge Check

5/5 answered

1. Which operator removes duplicate rows?

UNION ALL

1 UNION

JOIN

EXISTS

Correct\!

UNION removes duplicates; UNION ALL retains them.

2. What is a correlated subquery?

A query that runs without a table

1 A query dependent on the outer query

A query that combines tables

A query that removes duplicates

Correct\!

A correlated subquery references a column from its outer query.

3. Which keyword starts a CTE?

TEMP

DECLARE

2 WITH

CREATE

Correct\!

WITH introduces a Common Table Expression.

4. What does UNION ALL do?

Removes duplicates

1 Keeps duplicates

Sorts records

Joins tables

Correct\!

UNION ALL combines result sets without duplicate elimination.

5. What is a major benefit of multiple CTEs?

Deletes duplicate data

1 Improves readability of complex queries

Automatically creates indexes

Replaces every JOIN

Correct\!

Multiple CTEs let you break complex analysis into logical steps.

# 5/5

 Retake test

## 10. Day 17 Completion Checklist

- [ ]Understand UNION and UNION ALL.
- [ ]Write single-value and multiple-row subqueries.
- [ ]Understand correlated subqueries.
- [ ]Create CTEs using WITH.
- [ ]Write queries using multiple CTEs.
- [ ]Solve business analysis problems independently.

My recommendation: Spend extra practice time on CTEs and correlated subqueries. These are particularly useful when solving multi-step business analysis questions in SQL interviews.
