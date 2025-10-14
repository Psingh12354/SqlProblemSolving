# 📝 SQL Problem Solving Notes

This repository contains curated SQL problem-solving examples covering common real-world scenarios, including aggregations, joins, CASE expressions, and handling duplicates.

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

**Notes on CASE:**

* **WHEN** → specifies the condition
* **THEN** → specifies the result if the condition is true
* **ELSE** → optional, default result if no conditions match
* **END** → closes the CASE expression

---

## 2️⃣ Get Unique Sign-Up IDs for Transactions

**Problem:**
Retrieve a list of all `signup_id`s with transaction start dates in **April or May**. Each sign-up ID should appear **only once**.

```sql
SELECT DISTINCT signup_id
FROM transactions
WHERE transaction_start_date BETWEEN '2020-04-01' AND '2020-05-31';
```

**Notes:**

* `DISTINCT` ensures unique values.
* `BETWEEN` is inclusive of the start and end dates.

---

## 3️⃣ Find Customers Violating Primary Key Constraints

**Problem:**
Return all customers (`cust_id`) that appear **more than once** in the Customer Dimension (`dim_customer`). Output should include:

1. `cust_id`
2. Number of occurrences

```sql
SELECT cust_id, max_rn
FROM (
    SELECT cust_id,
          MAX(ROW_NUMBER() OVER(PARTITION BY cust_id ORDER BY cust_id)) OVER (PARTITION BY cust_id) AS max_rn
    FROM dim_customer
) AS t;
```

**Notes:**

* `ROW_NUMBER()` helps identify duplicates.
* `MAX(...) OVER (PARTITION BY ...)` counts the total occurrences.

---

## 4️⃣ Get Current Salary for Each Employee

**Problem:**
Some records are **outdated**, and salaries increase every year. Retrieve the **current salary** for each employee. Output:

* `id`
* `first_name`
* `last_name`
* `department_id`
* `current_salary`

Order the result by `employee ID` ascending.

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

**Notes:**

* `MAX(salary)` assumes salary increases yearly.
* `GROUP BY` ensures one row per employee.

---

## ✅ Tips for SQL Problem Solving

1. Always **alias columns** to avoid duplicate names when joining tables.
2. Use `CASE` for conditional logic inside SELECT or aggregation functions.
3. Use `DISTINCT` or `ROW_NUMBER()` to handle duplicates.
4. Prefer `MAX()` for current/latest value if data only increases over time.
5. For complex steps, consider **CTEs** for readability and modular queries.

---

