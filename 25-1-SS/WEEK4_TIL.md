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

### Concept
 `GROUP_CONCAT(expr)` returns a string result with the concatenated non-NULL values from a group. It returns NULL if there are no non-NULL values. The full **syntax** is as follows:

```sql
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val]) -- default: (,)
```

# 문제 풀이하기

## 문제1: 우유와 요거트가 담긴 장바구니
> GROUP_CONCAT()

### 요구사항
우유와 요거트를 동시에 구입한 장바구니의 아이디를 조회하는 SQL 문을 작성해주세요. 이때 결과는 장바구니의 아이디 순으로 나와야 합니다.

### 작성한 쿼리1
LIKE를 사용했기 때문에 만일 Milkshake와 같은 음식이 있다면 정확히 필터링 할 수 없다.
```sql
WITH MY AS (
    SELECT
        CART_ID,
        GROUP_CONCAT(NAME) AS NAME
    FROM CART_PRODUCTS
    GROUP BY CART_ID)

SELECT CART_ID
FROM MY
WHERE NAME LIKE '%Milk%'
    AND NAME LIKE '%Yogurt%'
ORDER BY CART_ID;
```

### 작성한 쿼리2: 쿼리1 보완(1)
```sql
WITH MY AS (
    SELECT
        CART_ID,
        CONCAT(',', GROUP_CONCAT(NAME), ',') AS NAME -- NAME 앞뒤로 쉼표로 구분해주면 LIKE를 사용하여 정확히 필터링 가능.
    FROM CART_PRODUCTS
    GROUP BY CART_ID)

SELECT CART_ID
FROM MY
WHERE NAME REGEXP ',Milk,'
  AND NAME REGEXP ',Yogurt,'
ORDER BY CART_ID;
```

### 작성한 쿼리3: 성능 좋은 쿼리(1)
```sql
SELECT CART_ID
FROM CART_PRODUCTS
WHERE NAME IN ('Milk', 'Yogurt')
GROUP BY CART_ID
HAVING COUNT(DISTINCT NAME) = 2
ORDER BY CART_ID;
```

### 작성한 쿼리4: 성능 좋은 쿼리(2)
```sql
SELECT CART_ID
FROM CART_PRODUCTS
WHERE NAME REGEXP '^Milk$|^Yogurt$' -- 정규표현식 사용
GROUP BY CART_ID
HAVING COUNT(DISTINCT NAME) = 2
ORDER BY CART_ID;
```

## 문제2: 언어별 개발자 분류하기
> GROUP_CONCAT()

### 요구사항
각 개발자가 보유한 기술 목록과 기술 카테고리를 요약하여 출력하는 쿼리를 작성해주세요.

### 작성한 쿼리
```sql
SELECT
    ID,
    EMAIL,
    GROUP_CONCAT(S.NAME) AS NAME,
    GROUP_CONCAT(S.CATEGORY) AS CATEGORY
FROM SKILLCODES AS S
JOIN DEVELOPERS AS D
ON (S.CODE & D.SKILL_CODE)=S.CODE
GROUP BY ID, EMAIL
```

## 문제3: 입양 시각 구하기(2)
> WITH RECURSIVE

### 요구사항
보호소에서는 몇 시에 입양이 가장 활발하게 일어나는지 알아보려 합니다. 0시부터 23시까지, 각 시간대별로 입양이 몇 건이나 발생했는지 조회하는 SQL문을 작성해주세요. 이때 결과는 시간대 순으로 정렬해야 합니다.

### 작성한 쿼리
```sql
WITH RECURSIVE T AS (
    SELECT 0 AS HOUR
    UNION
    SELECT HOUR+1 FROM T WHERE HOUR<23)
    
SELECT 
    T.HOUR,
    COALESCE(COUNT(A.DATETIME), 0) AS COUNT
FROM T
LEFT JOIN ANIMAL_OUTS A
ON T.HOUR = HOUR(A.DATETIME)
GROUP BY T.HOUR
ORDER BY T.HOUR;
```