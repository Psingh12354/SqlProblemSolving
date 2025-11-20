# ✅ **1) Average Rating Per City (Join Question)**

### **Problem Statement**

You are given two tables: **EmployeeDetails** and **EmployeeRatings**.
Write a SQL query to find the **average rating per city**.
Join the tables on *EmployeeID*.

### **Sample Tables**

**EmployeeDetails**

| employee_id | employee_name | city   |
| ----------- | ------------- | ------ |
| 101         | Arjun         | Mumbai |
| 102         | Priya         | Delhi  |
| 103         | Rohan         | Mumbai |

**EmployeeRatings**

| employee_id | rating |
| ----------- | ------ |
| 101         | 4      |
| 101         | 5      |
| 103         | 3      |

### **Solution**

```sql
SELECT 
    ed.city,
    AVG(er.rating) AS avg_rating
FROM EmployeeDetails ed
JOIN EmployeeRatings er
    ON ed.employee_id = er.employee_id
GROUP BY ed.city;
```

---

# ✅ **2) Employees With Salary Greater Than Department Average**

### **Sample Table: Employees**

| emp_id | emp_name | dept_name | salary |
| ------ | -------- | --------- | ------ |
| 1      | Arjun    | IT        | 60000  |
| 2      | Priya    | IT        | 75000  |
| 3      | Karan    | HR        | 50000  |
| 4      | Neha     | HR        | 70000  |

### **Solution**

```sql
SELECT e1.emp_name, e1.salary, e1.dept_name
FROM Employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM Employees e2
    WHERE e2.dept_name = e1.dept_name
);
```

---

# ✅ **3) Top 2 Salaries Per Department**

### **Sample Table: Employees**

| emp_id | name  | department | salary |
| ------ | ----- | ---------- | ------ |
| 1      | Arjun | IT         | 60000  |
| 2      | Priya | IT         | 75000  |
| 3      | Rohan | IT         | 72000  |
| 4      | Neha  | HR         | 80000  |
| 5      | Karan | HR         | 65000  |

### **Solution**

```sql
WITH ranked AS (
    SELECT 
        name,
        salary,
        department,
        DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS dr
    FROM Employees
)
SELECT name, salary, department
FROM ranked
WHERE dr <= 2;
```

---

# ✅ **4) Customers Who Didn’t Order in Last 30 Days**

### **Sample Tables**

**Customers**

| customer_id | customer_name |
| ----------- | ------------- |
| 101         | Amit          |
| 102         | Priya         |
| 103         | Rohan         |
| 104         | Sneha         |

**Orders**

| order_id | customer_id | order_date |
| -------- | ----------- | ---------- |
| 5001     | 101         | 2024-03-01 |
| 5002     | 102         | 2024-03-15 |
| 5003     | 103         | 2024-01-10 |

Assume today = **2024-03-31**

### **Solution**

```sql
SELECT c.customer_id, c.customer_name
FROM Customers c
LEFT JOIN Orders o
    ON c.customer_id = o.customer_id
    AND o.order_date >= CURRENT_DATE - INTERVAL '30' DAY
WHERE o.order_id IS NULL;
```

---

# ✅ **5) Running Total of Sales**

### **Sample Table: Sales**

| sale_date  | amount |
| ---------- | ------ |
| 2024-01-01 | 1000   |
| 2024-01-02 | 1500   |
| 2024-01-03 | 2000   |

### **Solution**

```sql
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM Sales;
```

---

# ✅ **6) Users Logged In for 3 Consecutive Days**

### **Sample Table: LoginHistory**

| user_id | login_date |
| ------- | ---------- |
| 101     | 2024-03-01 |
| 101     | 2024-03-02 |
| 101     | 2024-03-03 |
| 104     | 2024-03-15 |
| 104     | 2024-03-16 |
| 104     | 2024-03-17 |

### **Solution**

```sql
WITH t1 AS (
    SELECT
        user_id,
        login_date,
        LAG(login_date,1) OVER(PARTITION BY user_id ORDER BY login_date) AS prev1,
        LAG(login_date,2) OVER(PARTITION BY user_id ORDER BY login_date) AS prev2
    FROM LoginHistory
)
SELECT DISTINCT user_id
FROM t1
WHERE login_date = DATEADD(day, 1, prev1)
  AND prev1     = DATEADD(day, 1, prev2);
```

---

# ✅ **7) Delete Duplicates Keep Smallest ID**

### **Sample Table: Employees**

| emp_id | name  | city   |
| ------ | ----- | ------ |
| 1      | Arjun | Mumbai |
| 3      | Arjun | Mumbai |
| 4      | Sneha | Delhi  |
| 5      | Sneha | Delhi  |

### **Solution**

```sql
WITH dup AS (
    SELECT 
        emp_id,
        ROW_NUMBER() OVER (PARTITION BY name, city ORDER BY emp_id) AS rn
    FROM Employees
)
DELETE FROM dup
WHERE rn > 1;
```

---

# ✅ **8) Total Orders + Total Sales Per Month (2024)**

### **Sample Table: Orders**

| order_id | order_date | amount |
| -------- | ---------- | ------ |
| 1        | 2024-01-05 | 1000   |
| 2        | 2024-01-18 | 500    |
| 3        | 2024-02-03 | 1200   |

### **Solution**

```sql
SELECT
    FORMAT(order_date, 'yyyy-MM') AS month,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_sales
FROM Orders
WHERE YEAR(order_date) = 2024
GROUP BY FORMAT(order_date, 'yyyy-MM')
ORDER BY month;
```

---

# ✅ **9) Pivot Quarterly Sales**

### **Sample Table: QuarterlySales**

| product | Q1 | Q2 | Q3 | Q4 |
| ------- | -- | -- | -- | -- |
| Laptop  | 10 | 20 | 30 | 40 |

### **Solution**

```sql
SELECT product, quarter, sales
FROM QuarterlySales
UNPIVOT (
    sales FOR quarter IN (Q1, Q2, Q3, Q4)
) AS u;
```

---

# ✅ **10) Employee & Manager Names**

### **Sample Table: Employees**

| EmpID | Name  | ManagerID |
| ----- | ----- | --------- |
| 1     | Arjun | NULL      |
| 2     | Priya | 1         |
| 3     | Rohan | 1         |

### **Solution**

```sql
SELECT e.Name AS Employee, m.Name AS Manager
FROM Employees e
LEFT JOIN Employees m
    ON e.ManagerID = m.EmpID;
```

---

# ✅ **11) Employee Count Per Department (>5 only)**

### **Tables**

**Departments**

| dept_id | dept_name |
| ------- | --------- |
| 10      | IT        |
| 20      | HR        |

**Employees**

| emp_id | dept_id |
| ------ | ------- |
| 1      | 10      |
| 2      | 10      |
| 3      | 10      |
| 4      | 20      |
| 5      | 20      |
| 6      | 10      |

### **Solution**

```sql
SELECT d.dept_name, COUNT(e.emp_id) AS emp_count
FROM Departments d
JOIN Employees e
    ON d.dept_id = e.dept_id
GROUP BY d.dept_name
HAVING COUNT(e.emp_id) > 5;
```

---

# ✅ **12) Top-Selling Product Per Category**

### **Table: ProductSales**

| category    | product | sales |
| ----------- | ------- | ----- |
| Electronics | Laptop  | 5000  |
| Electronics | TV      | 7000  |
| Grocery     | Rice    | 2000  |
| Grocery     | Oil     | 1500  |

### **Solution**

```sql
WITH ranked AS (
    SELECT
        category,
        product,
        sales,
        RANK() OVER(PARTITION BY category ORDER BY sales DESC) AS rnk
    FROM ProductSales
)
SELECT category, product, sales
FROM ranked
WHERE rnk = 1;
```

---

# ✅ **13) First Order Date Per Customer**

### **Table: Orders**

| customer_id | order_date |
| ----------- | ---------- |
| 101         | 2024-01-05 |
| 101         | 2024-03-10 |
| 102         | 2024-02-20 |

### **Solution**

```sql
SELECT DISTINCT
    customer_id,
    FIRST_VALUE(order_date) 
       OVER(PARTITION BY customer_id ORDER BY order_date) AS first_order_date
FROM Orders;
```

---

# ✅ **14) Employees with Same Salary in Same Department**

### **Sample Table: Employees**

| emp_id | salary | department |
| ------ | ------ | ---------- |
| 1      | 50000  | IT         |
| 2      | 50000  | IT         |
| 3      | 60000  | HR         |
| 4      | 60000  | HR         |

### **Solution**

```sql
SELECT salary, department
FROM Employees
GROUP BY salary, department
HAVING COUNT(*) > 1;
```

---

# ✅ **15) Identify Schema Drift (Column Difference)**

### **Sample Tables**

**SourceTable Columns**

| column_name |
| ----------- |
| id          |
| name        |
| age         |
| city        |

**TargetTable Columns**

| column_name |
| ----------- |
| id          |
| name        |
| location    |

### **Solution**

```sql
SELECT column_name
FROM SourceTable
WHERE column_name NOT IN (SELECT column_name FROM TargetTable)
UNION ALL
SELECT column_name
FROM TargetTable
WHERE column_name NOT IN (SELECT column_name FROM SourceTable);
```
