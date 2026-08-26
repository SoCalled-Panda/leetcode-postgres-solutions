````markdown
# 184. Department Highest Salary

**Difficulty:** Medium · [Problem link](https://leetcode.com/problems/department-highest-salary/)

## Schema

Table: `Employee`

| Column       | Type    | Note                        |
|--------------|---------|-----------------------------|
| id           | int     | primary key                 |
| name         | varchar |                             |
| salary       | int     |                             |
| departmentId | int     | FK → `Department.id`        |

Table: `Department`

| Column | Type    | Note        |
|--------|---------|-------------|
| id     | int     | primary key |
| name   | varchar | NOT NULL    |

## Solution

```sql
SELECT department AS "Department",
       employee   AS "Employee",
       salary     AS "Salary"
FROM (
    SELECT d.name AS department,
           e.name AS employee,
           e.salary,
           DENSE_RANK() OVER (
               PARTITION BY e.departmentId
               ORDER BY e.salary DESC
           ) AS rnk
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
) ranked
WHERE rnk = 1;
```

## Explanation

Goal: *for each department*, find employee(s) with the **highest** salary.
This is a "**top-N per group**" problem → window functions shine here.

Step by step:

1. **JOIN** `Employee` ↔ `Department` to get the department name for each employee.
2. **`PARTITION BY departmentId`** restarts the ranking inside each department.
3. **`ORDER BY salary DESC`** ranks highest earners first.
4. Filter outer query to keep only rank `1` per partition.

⚠️ **Why `DENSE_RANK()` and not `ROW_NUMBER()`?** Ties!
In the example, Jim *and* Max both earn 90000 in IT — both must appear.
`ROW_NUMBER()` would assign them 1 and 2 randomly and drop one employee ❌.
Both `RANK()` and `DENSE_RANK()` label all tied top salaries as `1` ✅.

💡 **Bonus — aggregate + tuple `IN` version:**

```sql
SELECT d.name AS "Department",
       e.name AS "Employee",
       e.salary AS "Salary"
FROM Employee e
JOIN Department d ON e.departmentId = d.id
WHERE (e.departmentId, e.salary) IN (
    SELECT departmentId, MAX(salary)
    FROM Employee
    GROUP BY departmentId
);
```

Classic interview alternative: first compute each department's `MAX(salary)`
in a subquery, then match `(dept, salary)` pairs against it. Ties survive here
too, because *all* rows matching the max are returned.
````
