````markdown
# 181. Employees Earning More Than Their Managers

**Difficulty:** Easy · [Problem link](https://leetcode.com/problems/employees-earning-more-than-their-managers/)

## Schema

Table: `Employee`

| Column    | Type    | Note                          |
|-----------|---------|-------------------------------|
| id        | int     | primary key                   |
| name      | varchar |                               |
| salary    | int     |                               |
| managerId | int     | references another row's `id` |

Each employee's manager is *also a row in this same table*. Top-level managers (CEO) have `managerId = NULL`.

## Solution

```sql
SELECT e1.name AS "Employee"
FROM Employee e1
JOIN Employee e2
  ON e1.managerId = e2.id
WHERE e1.salary > e2.salary;
```

## Explanation

The employee and the manager live in the **same table**, so we join the table
with itself — a **self-join**.

By using two aliases we mentally split it into two roles:

- `e1` → the **employee**
- `e2` → the **manager**

The join condition `e1.managerId = e2.id` matches every employee with their own
manager's row. Then `WHERE e1.salary > e2.salary` keeps only those who earn more.

Two nice side effects of the INNER JOIN:

1. Rows where `managerId IS NULL` (the CEO) never match anything → dropped automatically, which is correct since they have no manager to compare against.
2. No explicit NULL filtering is needed.

💡 **Bonus — correlated subquery version:**

```sql
SELECT name AS "Employee"
FROM Employee e1
WHERE salary > (
    SELECT salary FROM Employee e2 WHERE e2.id = e1.managerId
);
```

Same logic, but the join happens inside the `WHERE`. Good to know both —
interviewers often ask how you'd rewrite one into the other.
````
