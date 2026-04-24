# 📝 SQL Problem Solving Notes

This repository contains curated SQL problem-solving examples covering common real-world scenarios — including **aggregations**, **joins**, **CASE expressions**, **handling duplicates**, and **date-based analytics**.

---

## 1️⃣ Calculate Difference Between Department Salaries

**Problem:**
Calculate the **difference between the highest salaries** in the `marketing` and `engineering` departments. Output **just the absolute difference**.

### Solution 1 – Using CTEs

```sql
WITH joinTable AS (
    SELECT 
        de.id AS emp_id,
        de.first_name,
        de.last_name,
        de.salary,
        dd.department
    FROM db_employee de
    JOIN db_dept dd 
      ON de.department_id = dd.id
),
maxOfMarketing AS (
    SELECT MAX(salary) AS markmaxSalary
    FROM joinTable
    WHERE department = 'marketing'
),
maxOfEngineering AS (
    SELECT MAX(salary) AS engmaxSalary
    FROM joinTable
    WHERE department = 'engineering'
)
SELECT 
    maxOfMarketing.markmaxSalary - maxOfEngineering.engmaxSalary AS salary_difference
FROM 
    maxOfMarketing, 
    maxOfEngineering;
```

### Solution 2 – Using CASE Expressions

```sql
SELECT 
    MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) -
    MAX(CASE WHEN d.department = 'engineering' THEN e.salary END) AS salary_difference
FROM db_employee e
JOIN db_dept d 
  ON e.department_id = d.id;
```

---

## 2️⃣ Get Unique Sign-Up IDs for Transactions

**Problem:**
Retrieve all `signup_id`s with transaction start dates in **April or May**. Each ID should appear **only once**.

```sql
SELECT DISTINCT signup_id
FROM transactions
WHERE transaction_start_date BETWEEN '2020-04-01' AND '2020-05-31';
```

---

## 3️⃣ Find Customers Violating Primary Key Constraints

**Problem:**
Return all customers (`cust_id`) that appear **more than once** in `dim_customer`.

```sql
SELECT cust_id, COUNT(*) AS occurrence_count
FROM dim_customer
GROUP BY cust_id
HAVING COUNT(*) > 1;
```

---

## 4️⃣ Get Current Salary for Each Employee

**Problem:**
Retrieve the **current salary** (latest value) for each employee.

```sql
SELECT 
    id,
    first_name,
    last_name,
    department_id,
    MAX(salary) AS current_salary
FROM ms_employee_salary
GROUP BY id, first_name, last_name, department_id
ORDER BY id;
```

---

## 5️⃣ Find Missing Data (NULL Check)

**Problem:**
Find all Airbnb search details where `host_response_rate` is missing.

```sql
SELECT *
FROM airbnb_search_details
WHERE host_response_rate IS NULL;
```

**Tip:**
To count missing records:

```sql
SELECT COUNT(*) AS missing_count
FROM airbnb_search_details
WHERE host_response_rate IS NULL;
```

---

## 6️⃣ Calculate Average Number of Likes per Post

**Problem:**
Find the average number of likes across all Facebook posts.

```sql
SELECT AVG(likes) AS average_likes
FROM facebook_posts;
```

**Note:**
`AVG()` ignores `NULL` values automatically.

---

## 7️⃣ Users with Both ‘Refinance’ and ‘InSchool’ Submissions

**Problem:**
Find all `user_id`s that have at least one `'Refinance'` and one `'InSchool'` submission.

```sql
SELECT user_id
FROM loans
WHERE type IN ('Refinance', 'InSchool')
GROUP BY user_id
HAVING COUNT(DISTINCT type) = 2;
```

---

## 8️⃣ Customer and Average Order Amount (Postmates Example)

**Problem:**
Find how many customers placed an order and the average order amount.

```sql
SELECT 
    COUNT(DISTINCT customer_id) AS total_customers,
    AVG(amount) AS average_order_amount
FROM postmates_orders;
```

**Alternative (if question says “how many orders”):**

```sql
SELECT 
    COUNT(*) AS total_orders,
    AVG(amount) AS average_order_amount
FROM postmates_orders;
```

---

## 9️⃣ Average Daily Active Users (DAU)

**Problem:**
Find the average **daily active users (DAU)** in January 2021 for each account.

```sql
SELECT 
    account_id,
    AVG(daily_active_users) AS avg_daily_active_users
FROM (
    SELECT 
        account_id,
        record_date,
        COUNT(DISTINCT user_id) AS daily_active_users
    FROM sf_events
    WHERE record_date BETWEEN '2021-01-01' AND '2021-01-31'
    GROUP BY account_id, record_date
) AS daily_counts
GROUP BY account_id;

---

## 10️⃣ Percentage of Orders Using Promotions

**Problem:** Calculate the percentage of orders in `online_orders` that used a promotion from `online_promotions`.

```sql
SELECT 
    ROUND(
        COUNT(CASE WHEN op.promotion_id IS NOT NULL THEN 1 END) * 100.0 / COUNT(*),
        2
    ) AS promotion_usage_percentage
FROM online_orders oo
LEFT JOIN online_promotions op
  ON oo.promotion_id = op.promotion_id;
```

**Notes:**

* `LEFT JOIN` ensures all orders are included, even if no promotion was used.
* `CASE WHEN op.promotion_id IS NOT NULL` counts only orders with valid promotions.
* Dividing by `COUNT(*)` and multiplying by 100 gives the **percentage of orders using promotions**.

---

## 11️⃣ Employees Hired Between January and July 2022

**Problem:** Find the number of employees hired between January and July 2022 inclusive.

```sql
SELECT 
    COUNT(*) AS num_of_emp
FROM employees
WHERE joining_date BETWEEN '2022-01-01' AND '2022-07-31';
```

**Alternative using YEAR and MONTH functions:**

```sql
SELECT 
    COUNT(*) AS num_of_emp
FROM employees
WHERE YEAR(joining_date) = 2022
  AND MONTH(joining_date) BETWEEN 1 AND 7;
```

**Notes:**

* `BETWEEN` includes both start and end dates.
* The alternative method uses `YEAR` and `MONTH` functions, which is useful if you want to ignore time portions in timestamps.

---

## 12️⃣ Average Session Duration by Session Type

**Problem:** Calculate the average session duration (in seconds) for each session type.

**MySQL Version:**

```sql
SELECT 
    session_type,
    ROUND(AVG(TIMESTAMPDIFF(SECOND, session_start, session_end)), 2) AS avg_session_duration_seconds
FROM twitch_sessions
GROUP BY session_type;
```

**PostgreSQL Version:**

```sql
SELECT 
    session_type,
    ROUND(AVG(EXTRACT(EPOCH FROM (session_end - session_start))), 2) AS avg_session_duration_seconds
FROM twitch_sessions
GROUP BY session_type;
```

**Notes:**

* `TIMESTAMPDIFF` (MySQL) and `EXTRACT(EPOCH FROM interval)` (PostgreSQL) calculate **duration in seconds**.
* `AVG()` calculates the **mean duration per session type**.
* `ROUND(..., 2)` ensures results are displayed with 2 decimal places.

---

## 13️⃣ Customers with Highest Daily Total Order Cost

**Problem:** Find customers with the highest daily total order cost between `2019-02-01` and `2019-05-01`.

```sql
WITH daily_totals AS (
    SELECT 
        c.first_name,
        o.order_date,
        SUM(o.total_order_cost) AS daily_total_cost
    FROM customers c
    JOIN orders o
      ON c.id = o.cust_id
    WHERE o.order_date BETWEEN '2019-02-01' AND '2019-05-01'
    GROUP BY c.first_name, o.order_date
)
SELECT first_name, order_date, daily_total_cost
FROM daily_totals
WHERE daily_total_cost = (
    SELECT MAX(daily_total_cost) FROM daily_totals
);
```

**Notes:**

* `SUM(total_order_cost)` aggregates all orders **per customer per day**.
* `WITH daily_totals AS (...)` creates a **CTE** to simplify finding the maximum daily total.
* The final `WHERE` clause ensures only the **customer(s) and date(s) with the highest daily total** are returned.

---

## 14️⃣ Running Total of Sales

**Problem:** Compute a running total of sales amount ordered by date.

```
WITH runningtot AS (
    SELECT 
        order_id,
        order_date,
        amount,
        SUM(amount) OVER (
            ORDER BY order_date 
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS rtotal
    FROM sales
)
SELECT order_id, order_date, amount, rtotal
FROM runningtot;
```

**Notes:**

- The SUM() OVER window function accumulates totals progressively.
- UNBOUNDED PRECEDING ensures the sum starts from the first record.
- Common in analytics for computing cumulative totals.

---

Got it 👍 — keeping it exactly in your existing format, no extra sections:

---

## 15️⃣ Repeated Logins Within Time Window

**Problem:** Count the number of repeated logins within 5 minutes of the previous login for the same user.

```sql
WITH prev_time_tab AS (
    SELECT 
        user_id,
        login_timestamp,
        LAG(login_timestamp) OVER (
            PARTITION BY user_id
            ORDER BY login_timestamp
        ) AS prev_time
    FROM logins
)
SELECT COUNT(*) AS repeated_login_count
FROM prev_time_tab
WHERE prev_time IS NOT NULL
  AND login_timestamp - prev_time <= INTERVAL '5 minutes';
```

**Notes:**

* `LAG()` is used to get the previous login timestamp for each user.
* Time difference is calculated using timestamp subtraction.
* `prev_time IS NOT NULL` ensures the first login is excluded.
* Only logins within 5 minutes of the previous login are counted.
----

---

Here you go — formatted exactly like your notes so you can **append as 16️⃣** 👇

---

## 16️⃣ Employee Hierarchy with Levels (Recursive CTE)

**Problem:** Retrieve all employees in a hierarchy starting from the top-level manager and assign a level to each employee based on their position in the hierarchy.

```sql
WITH RECURSIVE Manlevel AS (
    SELECT 
        emp_id, 
        emp_name, 
        manager_id, 
        1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT 
        e.emp_id, 
        e.emp_name, 
        e.manager_id, 
        m.level + 1
    FROM employees e
    JOIN Manlevel m
      ON e.manager_id = m.emp_id
)
SELECT * 
FROM Manlevel;
```

**Notes:**

* Anchor query starts with top-level managers (`manager_id IS NULL`).
* Recursive part joins employees with their managers to traverse hierarchy.
* `level` increments at each step to represent hierarchy depth.
* `UNION ALL` is used for performance (avoids unnecessary deduplication).
* Final result must be selected from the CTE (`Manlevel`), not the base table.

---

## ✅ Tips for SQL Problem Solving

1. **Alias columns/tables** for clarity.
2. Use **`CASE`** for conditional logic inside aggregations.
3. Use **`DISTINCT`** or **`ROW_NUMBER()`** for de-duplication.
4. Use **CTEs** (`WITH`) to make multi-step logic easier to read.
5. **`MAX()`**, **`AVG()`**, and **`COUNT()`** are powerful aggregation tools.
6. Always confirm whether a question refers to **rows (orders)** or **unique entities (customers)**.
7. For date-based questions, group by both **date** and **ID** before averaging.

---
