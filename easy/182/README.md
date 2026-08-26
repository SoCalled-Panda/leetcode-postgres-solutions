````markdown
# 182. Duplicate Emails

**Difficulty:** Easy · [Problem link](https://leetcode.com/problems/duplicate-emails/)

## Schema

Table: `Person`

| Column | Type    | Note                     |
|--------|---------|--------------------------|
| id     | int     | primary key              |
| email  | varchar | guaranteed NOT NULL here |

## Solution

```sql
SELECT email AS "Email"
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

## Explanation

A "duplicate email" = one email value occupying **more than one row**.
So: group the rows by email, then keep only groups whose size exceeds 1.

1. `GROUP BY email` — collapses identical emails into one group per address.
2. `HAVING COUNT(*) > 1` — discards singleton groups (emails seen once).
3. Each surviving group shrinks to a single output row → the result is
   naturally duplicate-free.

### 🪤 First attempt — kept as a lesson

```sql
SELECT email AS "Email"
FROM (
    SELECT email FROM person GROUP BY email
);
```

Looks plausible, but `GROUP BY email` alone answers *"which emails exist?"*
— it strips duplicates everywhere, including the legitimate ones, so `c@d.com`
leaks into the output. What was missing is a **group-level filter**: `HAVING`.
Once added, the wrapping subquery becomes dead weight — delete it.

> Rule of thumb: don't wrap a query in a subquery unless the wrapper
> contributes something (filtering, joining, ranking).

### 🔀 Bonus — self-join version

```sql
SELECT DISTINCT p1.email AS "Email"
FROM Person p1
JOIN Person p2
  ON p1.email = p2.email
 AND p1.id <> p2.id;
```

An email is duplicated ⟺ some **other** row (different `id`) holds the same
address. Every duplicate forms ≥ 2 matched pairs, so `DISTINCT` collapses the
pairs back to single values. Slower than `GROUP BY/HAVING` on big tables,
but great mental-model practice coming from #181's self-join.
````
