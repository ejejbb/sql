# Week0_TIL

## Subqueries

A subquery is a <U>SELECT</U> statement within another statement.

```sql
SELECT *
FROM t1
WHERE column1  -- outer query
    = (SELECT column1 FROM t2); -- subquery
```

We say that the subquery is **nested** within the outer query, and in fact it is possible to nest subqueries within other subqueries. A subquery must always appear within **parentheses(괄호)**

Many people find subqueries more readable than complex joins or unions. It gives us enough reason for studying subqueries.

A subquery can return a scalar, a single row, a single column, or a table.

A subquery's outer statement can be any one of: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `SET`, or `DO`.

### 15.2.15.1. The Subquery as Scalar Operand

- It is the simplest form.
- If a statement permits only a literal value, you cannot use a subquery. 

Ex1) `LIMIT`   
The `LIMIT` clause only expects a **literal value**, especially a numeric literal, not a dynamic result from a subquery.
```sql
[correct clause]
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5;
```
```sql
[⚠️wrong clause]
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT (SELECT COUNT(*) FROM departments);
```
Ex2) `DEFAULT`   
The `DEFAULT` value must also be a **literal** or a **function**, but not a subquery.
```sql
[correct clause]
CREATE TABLE users (
  id INT PRIMARY KEY,
  created_at TIMESTAMP DEFAULT NOW()
  );
-- NOW() is an allowed function
```
```sql
[⚠️wrong clause]
CREATE TABLE users (
  id INT PRIMARY KEY,
  some_column INT DEFAULT (SELECT MAX(id) FROM other_table);
-- subquery is not allowed for DEFAULT value
```

### 15.2.15.2. Comparisons Using Subqueries

Where comparison_operator is one of these operators:
```
=  >  <  >=  <=  <>  !=  <=>
```

An example of widely used Where comparison_operator:
```sql
... WHERE 'a' = (SELECT column1 FROM t1)
```

`LIKE` construction:
```SQL
non_subquery_operand LIKE (subquery)
```

An example of a common-form subquery comparison that you **cannot do with a join.**
```sql
SELECT *
FROM t1
  WHERE column1 = (SELECT MAX(column2) FROM t2);
-- It finds all the rows in table t1 for which the column1 value is equal to a maximum value in a table t2.
```
```sql
SELECT *
FROM t1 AS t
WHERE 2 = (SELECT COUNT(*)
           FROM t1
           WHERE t1.id = t.id);
-- It finds all rows in table t1 containing a value that occurs twice in a given column
```

### 15.2.15.3. Subqueries with ANY, IN or SOME

[`ANY` keyword]
- It returns *TRUE* if the comparison is *TRUE* for *ANY* of the values in the column that the subquery returns.
```sql
SELECT s1
FROM t1
WHERE s1 
    > ANY (SELECT s1 FROM t2);
-- Suppose that there is a row in table t1 containing (10).
-- TRUE if table t2 contains (21,14,7)
-- FALSE if talbe t2 contains (20,10)
-- unknown if talbe t2 contains (NULL, NULL, NULL)
```

- The word *IN* is an alias for *= ANY*. Thus, these two statements are the same:
```sql
SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);

SELECT s1 FROM t1 WHERE s1 IN    (SELECT s1 FROM t2);
```

- *IN* and *= ANY* are not synonyms when used with an **expression list**.
```sql
 -- [IN]: tuple comparison is available
SELECT *
FROM products
WHERE (category, price)
    IN (('Electronics', 1000), ('Books', 20));
```
```sql
 -- [ANY]: does not support tuple comparisons
 -- ⚠️wrong expression
WHERE (category, price)
    = ANY (SELECT category, price
           FROM discount_products);
```
```sql
 -- [ANY]: does not support tuple comparisons
 -- multi-column comparison is not allowed!
 -- ⚠️wrong expression
WHERE (category, price)
    = ANY (SELECT category, price
           FROM discount_products);
```
```sql
 -- [ANY]
 -- single-column comparison is allowed!
 -- correct expression
SELECT *
FROM products
WHERE price
    = ANY (SELECT discount_price
           FROM discount_products);
```

- *NOT IN* is not an alias for *<> ANY*, but for *<> ALL*.   
→ [15.2.15.4. Subqueries with ALL](#not-in-is-alias-for-all)

- The word *SOME* is an alias for *ANY*. Thus, these two statements are the same:
```sql
SELECT s1
FROM t1
WHERE s1
    <> ANY  (SELECT s1 FROM t2);

SELECT s1
FROM t1
WHERE s1
    <> SOME (SELECT s1 FROM t2);
-- use of the SOME is rare but it helps ensure that everyone understands the true meaning of the query.
-- "there is some b to which a is not equal"(o)
-- "a is not equal to any b"(x)
-- "there is no b which is equal to a"(x)
```

### 15.2.15.4. Subqueries with ALL

The word *ALL*, which must follow a comparison operator, means "return *TRUE* if the comparison is *TRUE* for *ALL* of the values in the column that the subquery returns."   
Example:
```sql
SELECT s1
FROM t1
WHERE s1
    > ALL (SELECT s1 FROM t2);
-- Suppose that there is a row in table t1 containing (10).
-- TRUE if table t2 contains (-5,-,5)
-- TRUE if table t2 is empty
-- FALSE if talbe t2 contains (12,6,NULL,-100)
-- unknown if talbe t2 contains (0,NULL,1)
```
If the table t2 is empty:
```sql
SELECT * FROM t1 WHERE 1 > ALL (SELECT s1 FROM t2);
-- TRUE

SELECT * FROM t1 WHERE 1 > (SELECT s1 FROM t2);
-- NULL

SELECT * FROM t1 WHERE 1 > ALL (SELECT MAX(s1) FROM t2);
-- NULL
```

#### *NOT IN* is alias for *<> ALL*

These two statements are the same:
```SQL
-- <> ALL
SELECT s1
FROM t1
WHERE s1
    <> ALL (SELECT s1 FROM t2);

-- NOT IN
SELECT s1
FROM t1
WHERE s1
    NOT IN (SELECT s1 FROM t2);
```

You can use TABLE with *ALL* and *NOT IN* if the following two conditions are met:
- The table in the subquery contains only **one column**
- The subquery does **not depend on a column expression**

### 15.2.15.5. Row Subqueries

A *row subquery* is a subquery variant that returns a single row and can thus return more than one column value.

Here are two examples:
```sql
SELECT *
FROM t1
WHERE (col1,col2)
    = (SELECT col3, col4
       FROM t2
       WHERE id = 10);

SELECT *
FROM t1
WHERE ROW(col1,col2)
    = (SELECT col3, col4
       FROM t2
       WHERE id = 10);

-- The two are equivalent.
-- The row constructor and the row returned by the subquery must contain the same number of values.
```

A row constructor is used for comparisons with subqueries that return two or more columns.
```sql
[⚠️wrong construction]
SELECT *
FROM t1
WHERE ROW(1)
    = (SELECT column1 FROM t2)
```

### 15.2.15.6. Subqueries with EXISTS or NOT EXISTS

If a subquery returns any rows at all, *EXISTS* subquery is *TRUE*, and *NOT EXISTS* subquery is *FALSE*.

Examples:
- What kind of store is present in one or more cities?   
하나 이상의 도시에 어떤 종류의 매장이 있습니까?
```sql
SELECT DISTINCT store_type
FROM stores s
WHERE EXISTS
    (SELECT *
    FROM cities_stores cs
    WHERE cs.store_type = s.store_type);
```
- What kind of store is present in no cities?   
어떤 종류의 상점이 어느 도시에도 없나요?
```sql
SELECT DISTINCT store_type
FROM stores s
WHERE NOT EXISTS
    (SELECT *
    FROM cities_stores cs
    WHERE cs.store_type = s.store_type);
```
- What kind of store is present in all cities?    
모든 도시에 어떤 종류의 상점이 있나요?
```sql
SELECT DISTINCT store_type
FROM stores s
WHERE NOT EXISTS (
    SELECT *
    FROM cities c
    WHERE NOT EXISTS (
      SELECT *
      FROM cities_stores cs
      WHERE cs.city = c.city
       AND cs.store_type = s.store_type));
```

### 15.2.15.7. Correlated Subqueries

A *correlated subquery* is a subquery that contains a **reference to a table** that also appears in the outer query.

For example:
```sql
SELECT *
FROM t1
WHERE column1
    = ANY (SELECT column1
           FROM t2
           WHERE t2.column2 = t1.column2);
```

### 15.2.15.10. Subquery Errors

⚠️Unsupported subquery syntax:
```sql
SELECT * 
FROM t1 
WHERE s1 IN (
    SELECT s2 
    FROM t2 
    ORDER BY s1 
    LIMIT 1
);
-- IN 서브쿼리 안에서 ORDER BY 절을 직접 사용할 수 없음.
```

⚠️Incorrect number of columns from subquery:
```sql
SELECT
    (SELECT column1, column2
    FROM t2)
FROM t1;
-- SELECT 절 안에서 서브쿼리를 값으로 사용할 경우, 서브쿼리는 반드시 단일 값을 반환해야 함.
```

⚠️Incorrect number of rows from subquery:
```SQL
SELECT *
FROM t1
WHERE column1
    = (SELECT column1 FROM t2);
-- t2에서 column1이 여러 개의 값을 반환할 경우 이러한 오류 발생
-- 여러 행을 `=`로 비교할 수 없음
```
```sql
[correct query]
SELECT *
FROM t1
WHERE column1
    = ANY (SELECT column1 FROM t2);
-- ANY를 사용할 경우, t2에서 column1이 여러 개의 값을 반환하더라도 사용 가능
```

### 15.2.15.11. Optimizing Subqueries

Move clauses from outside to inside the subquery.

Example1:
```sql
SELECT *
FROM t1
WHERE s1 IN (
    SELECT s1 FROM t1
    UNION ALL
    SELECT s1 FROM t2);
-- UNION ALL로 합친 결과에 대해 단일 검사
-- 데이터량이 많을수록 더 효율적
```
```sql
SELECT *
FROM t1
WHERE s1 IN (SELECT s1 FROM t1)
    OR s1 IN (SELECT s1 FROM t2);
-- 각각 IN을 별도 조건으로 검사
-- 데이터량이 많으면 성능 저하 가능성 있음
```

Example2:
```sql
SELECT
    (SELECT column1 + 5 FROM t1)
FROM t2;
-- 서브쿼리 내에서 +5
-- 서브쿼리 내에서 미리 계산
```
```sql
SELECT
    (SELECT column1 FROM t1) + 5
FROM t2;
-- 서브쿼리 결과에 +5
-- 메인쿼리에서 추가 연산 수행
-- 서브쿼리를 반복적으로 실행해야 할 수 있음
```

## CTE: `WITH`

A common table expression (CTE) is a named temporary result set that exists within the scope of a single statement and that can be referred to later within that statement, possibly multiple times.

To specify CTEs, use a `WITH` clause that has one or more comma-separated subclauses. Each subclause provides a subquery that produces a result set, and associates a name with the subquery.

Example:
```sql
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)

SELECT b, d
FROM cte1
JOIN cte2
WHERE cte1.a = cte2.c;
```

Determination of column names for a given CTE occurs as follows: These two queries are the same.

```sql
WITH cte (col1, col2) AS
(
  SELECT 1, 2
  UNION ALL
  SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```
```sql
WITH cte AS
(
  SELECT 1 AS col1, 2 AS col2
  UNION ALL
  SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```

A `WITH` clause is permitted in these contexts:
1. At the beginning of *SELECT, UPDATE*, and *DELETE* statements.
2. At the beginning of *subqueries*.

Only one `WITH` clause is permitted at the same level.
```sql
[⚠️illegal clause]
WITH cte1 AS (...)
WITH cte2 AS (...)
SELECT ...
```
```sql
[⚠️illegal clause]
WITH cte1 AS (...),
     cte1 AS (...)
SELECT ...
-- each CTE name must be unique to the clause
```
```sql
[legal clause]
WITH cte1 AS (...),
     cte2 AS (...)
SELECT ...
```

# 문제 풀이하기

## 문제1: 많이 주문한 테이블 찾기
> subquery

### 요구사항
식사 금액이 테이블 당 평균 식사 금액보다 더 많은 경우를 모두 출력하는 쿼리를 작성해주세요. 결과에는 tips 테이블에 있는 모든 컬럼이 포함되어야 합니다.

### 작성한 쿼리
```sql
SELECT *
FROM tips
WHERE total_bill >
  (SELECT AVG(total_bill)
  FROM tips)
```
![문제1](/image/image.png)

## 문제2: 레스토랑의 대목
> subquery

### 요구사항
요일별 매출액 합계를 구하고, 매출이 1500 달러 이상인 요일의 결제 내역을 모두 출력하는 쿼리를 작성해주세요. 쿼리 결과에는 tips 테이블에 있는 모든 컬럼이 포함되어야 합니다.

### 작성한 쿼리
조건에 해당하는 결제 내역을 **모두 출력**하기 위해서는 서브쿼리를 사용해야 한다.
```sql
SELECT *
FROM tips
WHERE day IN (
  SELECT day
  FROM tips
  GROUP BY day
  HAVING SUM(total_bill) > 1500
)
```
![문제2](/image/image2.png)


## 문제3: 식품분류별 가장 비싼 식품의 정보 조회하기
> subquery or WITH

### 요구사항
FOOD_PRODUCT 테이블에서 식품분류별로 가격이 제일 비싼 식품의 분류, 가격, 이름을 조회하는 SQL문을 작성해주세요. 이때 식품분류가 '과자', '국', '김치', '식용유'인 경우만 출력시켜 주시고 결과는 식품 가격을 기준으로 내림차순 정렬해주세요.

### 작성한 쿼리(subquery)
IN 절 안에 서브쿼리 사용

```sql
SELECT
    CATEGORY,
    PRICE AS MAX_PRICE,
    PRODUCT_NAME
FROM FOOD_PRODUCT
WHERE (CATEGORY, PRICE)
    IN (
        SELECT
            CATEGORY,
            MAX(PRICE)
        FROM FOOD_PRODUCT
        GROUP BY CATEGORY
    )
    AND CATEGORY IN ('과자', '국', '김치', '식용유')
GROUP BY CATEGORY
ORDER BY
    PRICE DESC;
```

### 작성한 쿼리(WITH)
가독성 향상

```sql
WITH CTE AS (
    SELECT
        CATEGORY,
        MAX(PRICE) AS MAX_PRICE
    FROM FOOD_PRODUCT
    GROUP BY CATEGORY
)
SELECT
    F.CATEGORY,
    F.PRICE AS MAX_PRICE,
    F.PRODUCT_NAME
FROM FOOD_PRODUCT F
JOIN CTE
    ON F.CATEGORY = CTE.CATEGORY
    AND F.PRICE = CTE.MAX_PRICE
WHERE F.CATEGORY IN ('과자', '국', '김치', '식용유')
ORDER BY F.PRICE DESC;
```

![문제3](/image/image3.png)