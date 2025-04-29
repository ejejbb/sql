# Week4_TIL

## 15.2.20. WITH (Common Table Expressions)

### Recursive Common Table Expressions

#### A recursive CTE is one having a subquery that refers to its own name.
```SQL
WITH RECURSIVE cte (column) AS
(
  -- Non-recursive part: return initial row set
  SELECT ...
  UNION ALL
  -- Recursive part: return additional row sets
  SELECT ... FROM cte_name WHERE condition
)
SELECT * FROM cte;
```
**How it works?**
- The `initial SELECT` produces the first set of rows.
- Then, the `recursive SELECT` keeps generating new rows based on the previous result set.
- Recursion ends when the stop condition (e.g., WHERE n < 5) is no longer satisfied.

**Details**
> **1) Each SELECT part can itself be a union of multiple SELECT statements.**
```sql
WITH RECURSIVE cte AS (
  SELECT 1 AS num
  UNION
  SELECT 2
  UNION ALL
  SELECT num + 1 FROM cte WHERE num < 4
)
SELECT * FROM cte;
```
Result
```
num
---
1
2
3
4
5
```

+Additional Example 1:
```sql
WITH RECURSIVE cte AS (
  SELECT 1 AS num
  UNION ALL
  SELECT 1
  UNION ALL
  SELECT num + 1 FROM cte WHERE num < 4
)
SELECT * FROM cte;
```
Result
```
num
---
1
1
2
2
3
3
4
4
```

+Additional Example 2:
```sql
WITH RECURSIVE cte AS (
  SELECT 1 AS num
  UNION ALL
  SELECT num + 1 FROM cte WHERE num < 4
)
SELECT * FROM cte;
```
Result
```
num
---
1
2
3
4
```


> **2) Column types for the CTE are determined only by the non-recursive (initial) SELECT.**
```sql
[⚠️wrong clause]
WITH RECURSIVE cte AS (
  SELECT 'abc' AS str
  UNION ALL
  SELECT CONCAT(str, str) FROM cte WHERE LENGTH(str) < 10
)
SELECT * FROM cte;
```
Result
```
str
---
abc
abc
abc
```
❗ Even though the string should grow (`abcabc`), it remains abc because the initial column type is too small (`CHAR(3)`).
```sql
[correct clause]
WITH RECURSIVE cte AS (
  SELECT CAST('abc' AS CHAR(20)) AS str
  UNION ALL
  SELECT CONCAT(str, str) FROM cte WHERE LENGTH(str) < 10
)
SELECT * FROM cte;
```
Result
```
str
---
abc
abcabc
abcabcabcabc
```

#### Columns are accessed by name, *not position*.   
Columns in the recursive part **can access columns in the nonrecursive part that have a different position**, as this CTE illustrates:
```sql
WITH RECURSIVE cte AS (
  SELECT
    1 AS n,
    1 AS p,
    -1 AS q
  UNION ALL
  SELECT
    n + 1,
    q * 2,
    p * 2
  FROM cte WHERE n < 5
)
SELECT * FROM cte;
```
Result
```
+------+------+------+
| n    | p    | q    |
+------+------+------+
|    1 |    1 |   -1 |
|    2 |   -2 |    2 |
|    3 |    4 |   -4 |
|    4 |   -8 |    8 |
|    5 |   16 |  -16 |
+------+------+------+
```

#### ⚠️The **recursive `SELECT`** part **must not contain** these constructs:
```
- Aggregate functions such as SUM()
- Window functions
- GROUP BY
- ORDER BY
- DISTINCT
```

#### The **recursive `SELECT`** must reference the **CTE only once** in the FROM clause.
```sql
[⚠️wrong clause: CTE inside recursive subquery]
WITH RECURSIVE cte AS (
  SELECT 1 AS n
  UNION ALL
  SELECT n + 1 FROM (SELECT * FROM cte) AS sub WHERE n < 5
)
SELECT * FROM cte;
```
→ ✅ You **can join other tables**, but the **CTE must not appear on the right-hand side of a LEFT JOIN**.

#### Practical Use Cases of Recursive CTEs

> a) Fibonacci Sequence
```sql
WITH RECURSIVE fibonacci (n, fib_n, next_fib_n) AS (
  SELECT 1, 0, 1
  UNION ALL
  SELECT n + 1, next_fib_n, fib_n + next_fib_n
  FROM fibonacci WHERE n < 10
)
SELECT * FROM fibonacci;
```
Result
```
| n | fib_n | next_fib_n |
|---|-------|------------|
| 1 | 0     | 1          |
| 2 | 1     | 1          |
| 3 | 1     | 2          |
| 4 | 2     | 3          |
| 5 | 3     | 5          |
| 6 | 5     | 8          |
| 7 | 8     | 13         |
| 8 | 13    | 21         |
| 9 | 21    | 34         |
|10 | 34    | 55         |
``` 
> b) Date Series Generation
```sql
WITH RECURSIVE dates (dt) AS (
  SELECT DATE('2023-01-01')
  UNION ALL
  SELECT dt + INTERVAL 1 DAY FROM dates
  WHERE dt + INTERVAL 1 DAY <= '2023-01-07'
)
SELECT * FROM dates;
```
Result
```
| dt          |
|-------------|
| 2023-01-01  |
| 2023-01-02  |
| 2023-01-03  |
| 2023-01-04  |
| 2023-01-05  |
| 2023-01-06  |
| 2023-01-07  |
```

> c) Hierarchical Tree Traversal (e.g., employee → manager path)

Data:
```
| id |   name   | manager_id |
|----|----------|------------|
| 333| Yasmina  | NULL       |
| 198| John     | 333        |
| 692| Tarek    | 333        |
| 29 | Pedro    | 198        |
|4610| Sarah    | 29         |
| 72 | Pierre   | 29         |
| 123| Adil     | 692        |
```

```sql
WITH RECURSIVE employee_path (id, name, path) AS (
  SELECT id, name, CAST(id AS CHAR(200))
  FROM employees WHERE manager_id IS NULL -- 초기값은 manager_id가 NULL인 가장 높은 직급의 사람
  UNION ALL
  SELECT e.id, e.name, CONCAT(ep.path, ',', e.id)
  FROM employee_path ep
  JOIN employees e ON ep.id = e.manager_id
)
SELECT * FROM employee_path ORDER BY path;
```
Result
```
| id   | name    | path            |
|------|---------|-----------------|
| 333  | Yasmina | 333             |
| 198  | John    | 333,198         |
| 29   | Pedro   | 333,198,29      |
| 4610 | Sarah   | 333,198,29,4610 |
| 72   | Pierre  | 333,198,29,72   |
| 692  | Tarek   | 333,692         |
| 123  | Adil    | 333,692,123     |
```


## 14.19.1 Aggregate Function Descriptions
> GROUP_CONCAT(expr)