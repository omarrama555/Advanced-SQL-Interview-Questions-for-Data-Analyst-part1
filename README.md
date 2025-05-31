
# 📊 Advanced SQL Interview Questions for Data Analyst

Below are 10 advanced SQL interview questions with sample answers tailored for a Data Analyst role. These questions test complex querying, performance optimization, and database concepts.

---

## 1. 🔁 Find and Remove Duplicate Records

**Question:** How would you write a SQL query to find duplicate records in a table and remove them while keeping the most recent record based on a timestamp?

**Answer:** Use `ROW_NUMBER()` with a CTE to rank and delete duplicates.

```sql
WITH RankedRecords AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY unique_column ORDER BY timestamp_column DESC) AS rn
    FROM table_name
)
DELETE FROM RankedRecords WHERE rn > 1;
```

---

## 2. 🔗 JOIN vs UNION vs UNION ALL

**Question:** Explain the difference between a JOIN, UNION, and UNION ALL. When would you use each?

**Answer:**

- **JOIN:** Combines rows from two or more tables based on a related column.
- **UNION:** Merges result sets vertically and removes duplicates.
- **UNION ALL:** Same as UNION but keeps duplicates for performance.

---

## 3. 📈 Running Total per Customer

**Question:** Write a SQL query to calculate the running total of sales for each customer over time.

```sql
SELECT 
    customer_id,
    order_date,
    sale_amount,
    SUM(sale_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS running_total
FROM sales
ORDER BY customer_id, order_date;
```

---

## 4. ⚡ Optimizing Slow Queries

**Question:** How do you optimize a SQL query that is running slowly on a large dataset?

**Answer:**
- Analyze the execution plan.
- Add indexes on filtered/joined columns.
- Avoid functions on indexed columns.
- Use query hints and rewrite joins.
- Partition large tables.
- Run `UPDATE STATISTICS`.

---

## 5. 🏆 Top 3 Products per Region

**Question:** Find the top 3 products by sales in each region for a given year, handling ties.

```sql
WITH RankedProducts AS (
    SELECT 
        region,
        product,
        SUM(sale_amount) AS total_sales,
        DENSE_RANK() OVER (PARTITION BY region ORDER BY SUM(sale_amount) DESC) AS sales_rank
    FROM sales
    WHERE YEAR(order_date) = 2025
    GROUP BY region, product
)
SELECT region, product, total_sales
FROM RankedProducts
WHERE sales_rank <= 3
ORDER BY region, sales_rank;
```

---

## 6. 🧱 Common Table Expressions (CTEs)

**Question:** What are CTEs and when would you use them instead of subqueries?

**Answer:**
- Improve readability in complex queries.
- Reusable within the same query.
- Useful for recursive queries.

```sql
WITH SalesByRegion AS (
    SELECT region, SUM(sale_amount) AS total_sales
    FROM sales
    GROUP BY region
)
SELECT region, total_sales
FROM SalesByRegion
WHERE total_sales > 100000;
```

---

## 7. 📅 Consecutive Monthly Purchases

**Question:** Find customers who made purchases in consecutive months.

```sql
WITH MonthlyPurchases AS (
    SELECT 
        customer_id,
        DATEPART(YEAR, order_date) AS purchase_year,
        DATEPART(MONTH, order_date) AS purchase_month,
        LAG(DATEPART(MONTH, order_date)) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_month
    FROM sales
)
SELECT DISTINCT customer_id
FROM MonthlyPurchases
WHERE purchase_month = prev_month + 1
   OR (purchase_month = 1 AND prev_month = 12 AND purchase_year = 
       LAG(purchase_year) OVER (PARTITION BY customer_id ORDER BY purchase_year, purchase_month) + 1);
```

---

## 8. 🚫 Handling NULLs in Aggregations

**Question:** How do you handle NULL values in SQL aggregations, and what are the implications?

**Answer:**
- Use `COALESCE` or `ISNULL` to replace NULLs.
- Use `NULLIF` to avoid divide-by-zero.
- Be cautious in joins — NULLs can exclude rows.

---

## 9. 📊 Pivoting Data by Quarter

**Question:** Pivot sales data by product category across quarters.

```sql
-- Using PIVOT
SELECT *
FROM (
    SELECT 
        product_category,
        DATEPART(QUARTER, order_date) AS quarter,
        sale_amount
    FROM sales
    WHERE YEAR(order_date) = 2025
) AS SourceTable
PIVOT (
    SUM(sale_amount)
    FOR quarter IN ([1], [2], [3], [4])
) AS PivotTable;

-- Using Conditional Aggregation
SELECT 
    product_category,
    SUM(CASE WHEN DATEPART(QUARTER, order_date) = 1 THEN sale_amount ELSE 0 END) AS Q1,
    SUM(CASE WHEN DATEPART(QUARTER, order_date) = 2 THEN sale_amount ELSE 0 END) AS Q2,
    SUM(CASE WHEN DATEPART(QUARTER, order_date) = 3 THEN sale_amount ELSE 0 END) AS Q3,
    SUM(CASE WHEN DATEPART(QUARTER, order_date) = 4 THEN sale_amount ELSE 0 END) AS Q4
FROM sales
WHERE YEAR(order_date) = 2025
GROUP BY product_category;
```

---

## 10. 🧠 Recursive Query for Hierarchical Data

**Question:** Implement a recursive query to get all employees under a manager.

```sql
WITH EmployeeHierarchy AS (
    SELECT employee_id, employee_name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.employee_name, e.manager_id, eh.level + 1
    FROM employees e
    INNER JOIN EmployeeHierarchy eh ON e.manager_id = eh.employee_id
)
SELECT employee_id, employee_name, manager_id, level
FROM EmployeeHierarchy
ORDER BY level, employee_id;
```

---

