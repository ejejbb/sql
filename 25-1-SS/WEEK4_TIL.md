# Week4_TIL

## 15.2.20. WITH (Common Table Expressions)

### Recursive Common Table Expressions

A recursive CTE is one having a subquery that refers to its own name.
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