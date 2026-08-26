````markdown
# 178. Rank Scores

**Difficulty:** Medium · [Problem link](https://leetcode.com/problems/rank-scores/)

## Schema

Table: `Scores`

| Column | Type    | Note        |
|--------|---------|-------------|
| id     | int     | primary key |
| score  | decimal | two decimals |

## Solution

```sql
SELECT score,
       DENSE_RANK() OVER (ORDER BY score DESC) AS "rank"
FROM Scores
ORDER BY score DESC;
```

## Explanation

Don't start with SQL — start by translating each *business rule* into a function behavior:

| Rule (from problem)                                        | Function trait needed       |
|------------------------------------------------------------|-----------------------------|
| Rank highest → lowest                                       | `ORDER BY score DESC`      |
| Ties share the same rank                                    | tied rows get equal value  |
| After a tie, next rank is **consecutive** (no gaps)         | never skips numbers        |

Only one function satisfies all three — `DENSE_RANK()`:

| score | ROW_NUMBER | RANK | DENSE_RANK ✅ |
|-------|-----------|------|--------------|
| 4.00  | 1         | 1    | 1            |
| 4.00  | 2         | 1    | 1            |
| 3.85  | 3         | 3    | 2            |
| 3.65  | 4         | 4    | 3            |
| 3.65  | 5         | 4    | 3            |
| 3.50  | 6         | 6    | 4            |

- `ROW_NUMBER()` ❌ invents fake distinctions between ties (1, 2).
- `RANK()` ❌ honors ties but then skips (…3rd place becomes 3, next is 5→here 6).
- `DENSE_RANK()` ✅ ties share, counting continues with no holes — exactly "dense".

Notes:

- **No `PARTITION BY`** — every row competes in one global league, so the
  window is just `OVER (ORDER BY ...)`.
- Final `ORDER BY score DESC` is presentation only (problem demands sorted
  output); it has no effect on the computed rank.
- 💡 **Demystified:** for any row,

  ```
  DENSE_RANK() = COUNT(DISTINCT scores strictly greater than mine) + 1
  ```

  Check it: score `3.65` has two distinct better scores `{4.00, 3.85}` → rank 3. ✔

### 🔀 Bonus — life before window functions (correlated subquery)

```sql
SELECT s1.score,
       (SELECT COUNT(DISTINCT s2.score)
        FROM Scores s2
        WHERE s2.score > s1.score) + 1 AS "rank"
FROM Scores s1
ORDER BY s1.score DESC;
```

The formula above, written out by hand. Works everywhere, O(n²)-ish —
a great "how would you do this without window functions?" interview answer,
and proof that `DENSE_RANK` isn't magic, just packaged math.
````
