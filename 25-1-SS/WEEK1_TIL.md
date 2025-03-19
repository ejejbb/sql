# Week1_TIL

## 14.20.2. Window Function Concepts and Syntax

A window function performs an aggregate-like operation on a set of query rows. However, whereas an **aggregate opertation** groups guery *rows into a single result row*, a **window function** produces *a result for each query row.*

<u>Example.</u>
```SQL
-- sales table
SELECT * FROM sales ORDER BY country, year, product;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2001 | Finland | Phone      |     10 |
| 2000 | India   | Calculator |     75 |
| 2000 | India   | Calculator |     75 |
| 2000 | India   | Computer   |   1200 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   1500 |
| 2001 | USA     | Computer   |   1200 |
| 2001 | USA     | TV         |    150 |
| 2001 | USA     | TV         |    100 |
+------+---------+------------+--------+
```

```SQL
-- aggregate operation
SELECT
    SUM(profit) AS total_profit
FROM sales;
+--------------+
| total_profit |
+--------------+
|         7535 |
+--------------+
SELECT
    country,
    SUM(profit) AS country_profit
FROM sales
GROUP BY country
ORDER BY country;
+---------+----------------+
| country | country_profit |
+---------+----------------+
| Finland |           1610 |
| India   |           1350 |
| USA     |           4575 |
+---------+----------------+
```

```SQL
-- window function
-- Window operations do not collapse groups of query rows to a single output row, but they produce a result for each row.
SELECT
    year, country, product, profit,
    SUM(profit) OVER() AS total_profit,
    SUM(profit) OVER(PARTITION BY country) AS country_profit
FROM sales
ORDER BY country, year, product, profit;
+------+---------+------------+--------+--------------+----------------+
| year | country | product    | profit | total_profit | country_profit |
+------+---------+------------+--------+--------------+----------------+
| 2000 | Finland | Computer   |   1500 |         7535 |           1610 |
| 2000 | Finland | Phone      |    100 |         7535 |           1610 |
| 2001 | Finland | Phone      |     10 |         7535 |           1610 |
| 2000 | India   | Calculator |     75 |         7535 |           1350 |
| 2000 | India   | Calculator |     75 |         7535 |           1350 |
| 2000 | India   | Computer   |   1200 |         7535 |           1350 |
| 2000 | USA     | Calculator |     75 |         7535 |           4575 |
| 2000 | USA     | Computer   |   1500 |         7535 |           4575 |
| 2001 | USA     | Calculator |     50 |         7535 |           4575 |
| 2001 | USA     | Computer   |   1200 |         7535 |           4575 |
| 2001 | USA     | Computer   |   1500 |         7535 |           4575 |
| 2001 | USA     | TV         |    100 |         7535 |           4575 |
| 2001 | USA     | TV         |    150 |         7535 |           4575 |
+------+---------+------------+--------+--------------+----------------+
```

The **OVER clause** is permitted for many <u>aggregate functions</u>, which therefore can be used as window or nonwindow functions, *depending on whether the OVER clause is present or absent*:
```sql
AVG()
COUNT()
MAX()
MIN()
SUM()
BIT_AND()
BIT_OR()
BIT_XOR()
JSON_ARRAYAGG()
JSON_OBJECTAGG()
STDDEV_POP(), STDDEV(), STD()
STDDEV_SAMP()
VAR_POP(), VARIANCE()
VAR_SAMP()
```

MySQL also supports <u>nonaggregate functions</u> that are used only as window functions. For these, the **OVER clause is mandatory**:
```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
PERCENT_RANK()
LAG()
LEAD()
NTILE()
FIRST_VALUE()
LAST_VALUE()
NTH_VALUE()
CUME_DIST()
```

The **OVER clause** has <u>two forms</u>:
```sql
over_clause:
    {OVER (window_spec) | OVER window_name}
```
Case1. **OVER (window_spec)**   
The **window specification** appears directly in the OVER clause, between the parentheses.
```sql
window_spec:
    [window_name] [partition_clause] [order_clause] [frame_clause]
-- PARTITION BY: indicates how to divide the query rows into groups
-- ORDER BY: indicates how to sort rows in each partition
-- FRAME: specifies how to define the subset
```

Case2. **OVER window_name**   
*window_name* is the name for a window specification defined by a WINDOW clause elsewhere in the query.

**14.20.4. Named Windows**   
Windows can be defined and names by which to refer to them in *OVER clause.*

1) reduce multiple windows
```sql
-- define (ORDER BY val) multiple times
SELECT
  val,
  ROW_NUMBER() OVER (ORDER BY val) AS 'row_number',
  RANK()       OVER (ORDER BY val) AS 'rank',
  DENSE_RANK() OVER (ORDER BY val) AS 'dense_rank'
FROM numbers;
```
```sql
-- simplify by using WINDOW to define the window once and referring to the window by name in the OVER clauses
SELECT
  val,
  ROW_NUMBER() OVER w AS 'row_number',
  RANK()       OVER w AS 'rank',
  DENSE_RANK() OVER w AS 'dense_rank'
FROM numbers
WINDOW w AS (ORDER BY val);
```

2) the named window can be modified by the addition of other clauses
```sql
-- apply different ORDER BY specification
SELECT
  DISTINCT year, country,
  FIRST_VALUE(year) OVER (w ORDER BY year ASC) AS first,
  FIRST_VALUE(year) OVER (w ORDER BY year DESC) AS last
FROM sales
WINDOW w AS (PARTITION BY country);
```

3) Rules   
✔️ Only adding properties to a named window is possible.(modify❌)   
✔️ Forward and backward references are permitted, but not cycles.

## 14.20.1. Window Function Descriptions
> ☀️ *Over* clause _ Nonaggregate window function

**TABLE. Window Functions**
|Name|Description|
|----|-----------|
|ROW_NUMBER()|중복없이 해당 행의 순번 부여|
|RANK()|동점이 있을 경우 다음 순위 건너뜀|
|DENSE_RANK()|동점이 있어도 순위를 건너뛰지 않음|
|LAG()|행 기준 비교할 때 이전 행의 값을 참조|
|LEAD()|미래 값과 비교, 다음 값과 차이 계산|
|NTILE(N)|Value of argument from N-th row of window frame|
|FIRST_VALUE()|Value of argument from first row of window frame|
|LAST_VALUE()|Value of argument from last row of window frame|

1. ROW_NUMBER(): 중복 없이 순번 부여
- rows numbers range from 1 to the number of partition rows
- `ORDER BY` affects the order in which rows are numbered
- assigns peers different row numbers

```sql
SELECT
    year, country, product, profit,
    ROW_NUMBER() OVER (PARTITION BY country) AS row_num1, -- ORDER BY를 사용하지 않는 경우 정렬 순서 불명확
    ROW_NUMBER() OVER (PARTITION BY country ORDER BY year, product) AS row_num2 -- year, product 기준 오름차순 정렬
FROM sales;
+------+---------+------------+--------+----------+----------+
| year | country | product    | profit | row_num1 | row_num2 |
+------+---------+------------+--------+----------+----------+
| 2000 | Finland | Computer   |   1500 |        2 |        1 |
| 2000 | Finland | Phone      |    100 |        1 |        2 |
| 2001 | Finland | Phone      |     10 |        3 |        3 |
| 2000 | India   | Calculator |     75 |        2 |        1 |
| 2000 | India   | Calculator |     75 |        3 |        2 |
| 2000 | India   | Computer   |   1200 |        1 |        3 |
| 2000 | USA     | Calculator |     75 |        5 |        1 |
| 2000 | USA     | Computer   |   1500 |        4 |        2 |
| 2001 | USA     | Calculator |     50 |        2 |        3 |
| 2001 | USA     | Computer   |   1500 |        3 |        4 |
| 2001 | USA     | Computer   |   1200 |        7 |        5 |
| 2001 | USA     | TV         |    150 |        1 |        6 |
| 2001 | USA     | TV         |    100 |        6 |        7 |
+------+---------+------------+--------+----------+----------+
```

2. RANK(): 중복 순번 부여, 다음 순번 건너뜀
- peers are considered ties and receive the same rank
- it does not assign consecutive ranks to peer groups if groups of size grater than one exist
- should be used with `ORDER BY`

3. DENSE_RANK(): 중복 순번 부여, 다음 순번 건너뛰지 않음
- peers are considered ties and receive the same rank
- it assigns consecutive ranks to peer groups

4. LAG(): 이전 행 참조
- returns the value of expr from the row that precedes the current row by N rows within its partition (N:positive integer)
- often used to compute differences between rows

5. LEAD(): 이후 행 참조
- returns the value of expr from the row that follows the current row by N rows within its partition
- often used to compute differences between rows

```SQL
SELECT
    t, val,
    LAG(val)        OVER w AS 'lag', -- 현재 행보다 한 행 이전의 값
    LEAD(val)       OVER w AS 'lead', -- 현재 행보다 한 행 이후의 값
    val - LAG(val)  OVER w AS 'lag diff', -- 현재 값과 이전 값의 차이
    val - LEAD(val) OVER w AS 'lead diff' -- 현재 값과 이후 값의 차이
FROM series
WINDOW w AS (ORDER BY t);
+----------+------+------+------+----------+-----------+
| t        | val  | lag  | lead | lag diff | lead diff |
+----------+------+------+------+----------+-----------+
| 12:00:00 |  100 | NULL |  125 |     NULL |       -25 |
| 13:00:00 |  125 |  100 |  132 |       25 |        -7 |
| 14:00:00 |  132 |  125 |  145 |        7 |       -13 |
| 15:00:00 |  145 |  132 |  140 |       13 |         5 |
| 16:00:00 |  140 |  145 |  150 |       -5 |       -10 |
| 17:00:00 |  150 |  140 |  200 |       10 |       -50 |
| 18:00:00 |  200 |  150 | NULL |       50 |      NULL |
+----------+------+------+------+----------+-----------+
```

```SQL
-- LAG(n, 1, 0): n=expression(조회할 컬럼 또는 값), 1=offset(1행 이전 값을 참조), 0=default(이전 값이 없으면 0 반환)
-- LEAD(n, 1, 0): n=expression(조회할 컬럼 또는 값), 1=offset(1행 이후 값을 참조), 0=default(이후 값이 없으면 0 반환)
SELECT
    n, -- 현재 행의 값
    LAG(n, 1, 0)      OVER w AS 'lag', -- 현재 행 기준으로 이전 행의 값. 이전 값이 없으면 기본값 0 반환
    LEAD(n, 1, 0)     OVER w AS 'lead', -- 현재 행 기준으로 다음 행의 값. 다음 값이 없으면 기본값 0 반환
    n + LAG(n, 1, 0)  OVER w AS 'next_n', -- 현재 값과 이전 값을 더한 값 = 피보나치 수열의 다음 수
    n + LEAD(n, 1, 0) OVER w AS 'next_next_n' -- 현재 값과 다음 값을 더한 값 = 피보나치 수열의 다다음 수
    FROM fib
    WINDOW w AS (ORDER BY n);
+------+------+------+--------+-------------+
| n    | lag  | lead | next_n | next_next_n |
+------+------+------+--------+-------------+
|    1 |    0 |    1 |      1 |           2 |
|    1 |    1 |    2 |      2 |           3 |
|    2 |    1 |    3 |      3 |           5 |
|    3 |    2 |    5 |      5 |           8 |
|    5 |    3 |    8 |      8 |          13 |
|    8 |    5 |    0 |     13 |           8 |
+------+------+------+--------+-------------+
```

6. NTILE(N): N 등분
- devides a partition into N groups (N:positive integer)
- should be used with `ORDER BY` to sort partition rows into the desired order

7. FIRST_VALUE()
- returns the value of expr from the **first row** of the window frame

8. LAST_VALUE()
- returns the value of expr from the **last row** of the window frame

## 14.19.1. Aggregate Function Descriptions
> ☀️ *Over* clause _ Aggregate window function

- Aggregate fucations can be used as a window function when the `over_clause` is added.
- The `DISTINCT` option is not allowed in most cases when using aggregate functions as window functions (e.g., `AVG`, `SUM`, `MAX`, `MIN`).
- Window functions define their window range by combining `PARTITION BY`, `ORDER BY`, and `FRAME` clauses within the `OVER()` clause.
- Window functions can only be used in the `SELECT` and `ORDER BY` clauses.

| Name | Description |
|-------|------|
| AVG() | return the average value of expr |
| COUNT() | return a count of the number of rows returned |
| MAX() | return maximum value |
| MIN() | return minimum value |
| SUM() | return the sum |

# 문제 풀이하기

## 문제1: Rank Scores
> window function: DENSE_RANK()

### 요구사항
Write a solution to find the rank of the scores. The ranking should be calculated according to the following rules:

- The scores should be ranked from the highest to the lowest.
- If there is a tie between two scores, both should have the same ranking.
- After a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no holes between ranks.
Return the result table ordered by score in descending order.

### 작성한 쿼리
MYSQL에서 `RANK`는 예약어이기 때문에 예약어를 별칭으로 쓰려면 백틱으로 감싸야 한다.
```sql
SELECT
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS `rank`
FROM Scores;
```
![문제1](/image/image4.png)

## 문제2: 다음날도 서울숲의 미세먼지 농도는 나쁨 😢
> window function: LEAD()

### 요구사항
당일의 미세먼지 농도보다 바로 다음날의 미세먼지 농도가 더 안좋은 날을 찾아주세요.

### 작성한 쿼리
- *OVER clause* is mandatory for nonaggregate window function.
- Cannot use window functions directly in the WHERE clause.
→ Window functions create values in the SELECT clause, and you need to wrap them in a **subquery or a CTE (Common Table Expression)** to filter them.
```sql
WITH CTE AS (
  SELECT
    measured_at AS today,
    LEAD(measured_at) OVER (ORDER BY measured_at) AS next_day,
    pm10,
    LEAD(pm10) OVER (ORDER BY measured_at) AS next_pm10
  FROM measurements
)

SELECT *
FROM CTE
WHERE pm10 < next_pm10;
```

### 작성한 쿼리2
```sql
-- simplify by using WINDOW to define the window once and referring to the window by name in the OVER clauses
WITH CTE AS (
  SELECT
    measured_at AS today,
    LEAD(measured_at) OVER w AS next_day,
    pm10,
    LEAD(pm10) OVER w AS next_pm10
  FROM measurements
  WINDOW w AS (ORDER BY measured_at)
)
SELECT *
FROM CTE
WHERE pm10 < next_pm10;
```
![문제2](/image/image5.png)


## 문제3: 그룹별 조건에 맞는 식당 목록 출력하기
> window function: COUNT()+over_clause, RANK()

### 요구사항
MEMBER_PROFILE와 REST_REVIEW 테이블에서 리뷰를 가장 많이 작성한 회원의 리뷰들을 조회하는 SQL문을 작성해주세요. 회원 이름, 리뷰 텍스트, 리뷰 작성일이 출력되도록 작성해주시고, 결과는 리뷰 작성일을 기준으로 오름차순, 리뷰 작성일이 같다면 리뷰 텍스트를 기준으로 오름차순 정렬해주세요.

### 작성한 쿼리
```SQL
WITH RR AS (
    SELECT
        *,
        RANK() OVER (ORDER BY CNT DESC) AS RK
    FROM (
        SELECT
            MEMBER_ID,
            REVIEW_TEXT,
            REVIEW_DATE,
            COUNT(*) OVER (PARTITION BY MEMBER_ID) AS CNT
        FROM REST_REVIEW) AS R
    )

SELECT
    MEMBER_NAME,
    REVIEW_TEXT,
    DATE_FORMAT(REVIEW_DATE, '%Y-%m-%d') AS REVIEW_DATE
FROM MEMBER_PROFILE AS M
JOIN RR
USING (MEMBER_ID)
WHERE RK=1
ORDER BY REVIEW_DATE, REVIEW_TEXT;
```