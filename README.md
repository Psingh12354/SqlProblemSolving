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
```

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
