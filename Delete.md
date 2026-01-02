---

# 📘 SQL DELETE – Interview Notes (with Queries)

---

## 🔹 1️⃣ Delete Duplicate Records (Keep Latest)

### 📌 Problem

Delete duplicate records and keep only the **latest record per entity**.

---

### ✅ Query (Interview-Safe & Portable)

```sql
WITH cte AS (
    SELECT emp_id,
           updated_at,
           ROW_NUMBER() OVER (
               PARTITION BY emp_id
               ORDER BY updated_at DESC
           ) AS rn
    FROM employee_salary
)
DELETE FROM employee_salary e
WHERE EXISTS (
    SELECT 1
    FROM cte c
    WHERE e.emp_id = c.emp_id
      AND e.updated_at = c.updated_at
      AND c.rn > 1
);
```

---

### 🧠 What to Explain

* `ROW_NUMBER()` identifies duplicates
* `rn = 1` → latest record
* `rn > 1` → duplicates to delete
* DELETE happens on **base table**

---

### 🏆 Interview One-liner

> “I rank records using ROW_NUMBER and delete rows where rank is greater than one.”

---

---

## 🔹 2️⃣ Delete Records Using Another Table

### 📌 Problem

Delete records from main table that do **not exist** in a reference table.

---

### ✅ Query (NULL-Safe & Preferred)

```sql
DELETE FROM employee_records e
WHERE NOT EXISTS (
    SELECT 1
    FROM valid_departments v
    WHERE e.dept_id = v.dept_id
);
```

---

### 🧠 What to Explain

* Deletes unmatched records
* `NOT EXISTS` avoids NULL issues
* Safer than `NOT IN`

---

### 🏆 Interview One-liner

> “I prefer NOT EXISTS over NOT IN to avoid NULL-related issues.”

---

---

## 🔹 3️⃣ Delete Records Using Aggregate Condition

### 📌 Problem

Delete **all records** of customers whose **total transaction amount < 200**.

---

### ✅ Query (Correct Logic)

```sql
DELETE FROM customer_transactions
WHERE customer_id IN (
    SELECT customer_id
    FROM customer_transactions
    GROUP BY customer_id
    HAVING SUM(txn_amount) < 200
);
```

---

### ✅ Alternative (Production-Safe)

```sql
DELETE FROM customer_transactions ct
WHERE EXISTS (
    SELECT 1
    FROM customer_transactions c
    WHERE ct.customer_id = c.customer_id
    GROUP BY c.customer_id
    HAVING SUM(c.txn_amount) < 200
);
```

---

### 🧠 What to Explain

* Aggregates work at **group level**
* Identify group keys first
* Delete all rows of that group

---

### 🏆 Interview One-liner

> “For aggregate-based deletes, I identify group keys using HAVING and delete all related rows.”

---

---

## 🔥 Common Interview Mistakes (Say You Avoid These)

* ❌ `DELETE FROM CTE`
* ❌ Using aggregates directly in WHERE
* ❌ Using `NOT IN` with NULLs
* ❌ Missing deterministic ORDER BY
* ❌ Deleting without validation

---

## 🧠 Final Interview Tip (Very Important)

Always say:

```sql
SELECT * FROM table
-- same WHERE condition
```

before running DELETE in production.

---

## ✅ 30-Second Interview Summary (Memorize This)

> “For deletes, I always identify rows first using CTEs or subqueries, validate with SELECT, and then delete from the base table using EXISTS for safety and portability.”

---
