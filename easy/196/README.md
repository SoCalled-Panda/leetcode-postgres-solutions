````markdown
# 196. Delete Duplicate Emails

**Difficulty:** Easy · [Problem link](https://leetcode.com/problems/delete-duplicate-emails/)

## Schema

Table: `Person`

| Column | Type    | Note        |
|--------|---------|-------------|
| id     | int     | primary key |
| email  | varchar |             |

## Solution

```sql
DELETE FROM Person
WHERE id NOT IN (
    SELECT MIN(id)
    FROM Person
    GROUP BY email
);
```

## Explanation

Key insight: instead of thinking *"which duplicates do I remove?"*,
think inverted — **"which rows must survive?"**

Step by step:

1. The inner subquery lists every email's **survivor id** — its `MIN(id)`
   (`GROUP BY email` makes `MIN` run once per distinct email).
2. The outer `DELETE` removes every row whose `id` is **not** in that set.

Why the subquery at all? Because `WHERE id <> MIN(id)` is illegal —
aggregates can't appear in a plain `WHERE`. We need `GROUP BY`/`HAVING`
context first, and a subquery provides exactly that.

💡 **Pro habit:** before writing DELETE, preview the victims:

```sql
SELECT * FROM Person      -- change DELETE back temporarily
WHERE id NOT IN (SELECT MIN(id) FROM Person GROUP BY email);
```

Verify those are exactly the rows you intend to kill, *then* switch to
`DELETE`. Cheap insurance when the table holds real data.

### 🔀 Bonus 1 — Postgres-only self-join delete (`USING`)

```sql
DELETE FROM Person p
USING Person q
WHERE p.email = q.email
  AND p.id > q.id;
```

Reads almost like English: *"delete me if a row with the same email exists
and has a smaller id."* `DELETE ... USING` is a PostgreSQL extension —
the cross-table FROM clause for deletions — and your repo being
postgres-specific is the perfect excuse to show it off.

### 🔀 Bonus 2 — NULL-safe anti-join (`NOT EXISTS`)

```sql
DELETE FROM Person p
WHERE NOT EXISTS (
    SELECT 1 FROM Person q
    WHERE q.email = p.email
      AND q.id < p.id
);
```

A row survives **iff no smaller-id twin exists** — which by definition keeps
exactly one row per email. Bonus: unlike `NOT IN`, `NOT EXISTS` is immune
to the NULL poison problem and often planner-friendlier on big tables.
````
