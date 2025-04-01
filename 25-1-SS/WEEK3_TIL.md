# Week3_TIL

## 14.5 Flow Control Functions

### Table 14.7 Flow Control Operators

|Name|	Description|
|--|--|
|CASE|	Case operator|
|IF()|	If/else construct|
|IFNULL()|	Null if/else construct|
|NULLIF()|	Return NULL if expr1 = expr2|

### 1. CASE

```SQL
-- 1. returns the result for the first `value=compare_value` comparison that is true
CASE value WHEN compare_value THEN result [WHEN compare_value THEN result ...] [ELSE result] END

SELECT
    CASE 1
    WHEN 1 THEN 'one'
    WHEN 2 THEN 'two'
    ELSE 'more' END;
-> 'one'

-- 2. returns the result for the first condition that is true
CASE WHEN condition THEN result [WHEN condition THEN result ...] [ELSE result] END

SELECT
    CASE WHEN 1>0 THEN 'true'
    ELSE 'false' END;
-> 'true'
```

### 2. IF(expr1, expr2, expr3)

If `expr1` is **TRUE** (expr1 <> 0 and expr1 IS NOT NULL), it returns `expr2`. Otherwise, it returns `expr3`.

```sql
SELECT IF(1>2,2,3);
-- FALSE -> 3
SELECT IF(1<2,'yes','no');
-- TRUE -> 'yes'
```

### 3. IFNULL(expr1,expr2)

If `expr1` is not NULL, it returns `expr1`; otherwise it returns `expr2`.

```sql
SELECT IFNULL(1,0);
-- not NULL -> 1
mysql> SELECT IFNULL(NULL,10);
-- NULL -> 10
mysql> SELECT IFNULL(1/0,10);
-- INFINITE(NULL) -> 10
mysql> SELECT IFNULL(1/0,'yes');
-- INFINITE(NULL) -> 'yes'
```

### 4. NULLIF(expr1,expr2)

Returns `NULL` if `expr1 = expr2 is true`, otherwise returns `expr1`. This is the same as `CASE WHEN expr1 = expr2 THEN NULL ELSE expr1 END`.

```sql
SELECT NULLIF(1,1);
-- 1=1 -> NULL
SELECT NULLIF(1,2);
-- 1!=2 -> 1
```

### Type of returned value

The return type is determined by the more general type (**STRING > REAL > INTEGER**).

> **Note**   
The result of functions like **CASE** and **IF()** may be stored in temporary tables, so the return type is important.   
Comparisons between values with different character sets may result in errors. In such cases, **use CAST() to explicitly convert character sets**.

<u>Example</u>
```sql
SELECT
    IFNULL(1, 'test') AS result;

-- 1 is INT, 'test' is STRING
-- result: 1
-- If this result is stored in a temporary table, the number 1 will also be stored as a STRING.
```

## 14.4.2 Comparison Functions and Operators

### Table 14.4 Comparison Operators

|Name|	Description|
|--|--|
|<>, !=|	Not equal operator|
|<=>|	NULL-safe equal to operator|
|BETWEEN ... AND ...|	Whether a value is within a range of values|
|NOT BETWEEN ... AND ...|	Whether a value is not within a range of values|
|COALESCE()|	Return the first non-NULL argument|
|EXISTS()|	Whether the result of a query contains any rows|
|NOT EXISTS()|	Whether the result of a query contains no rows|
|GREATEST()|	Return the largest argument|
|LEAST()|	Return the smallest argument|
|IN()|	Whether a value is within a set of values|
|NOT IN()|	Whether a value is not within a set of values|
|INTERVAL()|	Return the index of the argument that is less than the first argument|
|IS|	Test a value against a boolean|
|IS NOT|	Test a value against a boolean|
|IS NOT NULL|	NOT NULL value test|
|IS NULL|	NULL value test|
|ISNULL()|	Test whether the argument is NULL|
|LIKE|	Simple pattern matching|
|NOT LIKE|	Negation of simple pattern matching|
|STRCMP()|	Compare two strings|

### 1. Basic Comparison Operators

|Name|	Description|
|--|--|
|<>, !=|	Not equal operator|
|<=>|	NULL-safe equal to operator|

```SQL
SELECT
    1 <=> 1, -- 1
    NULL <=> NULL, -- 1
    1 <=> NULL; -- 0
```

### 2. Range and Set Comparisons

|Name|	Description|
|--|--|
|BETWEEN ... AND ...|	Whether a value is within a range of values|
|NOT BETWEEN ... AND ...|	Whether a value is not within a range of values|
|EXISTS()|	Whether the result of a query contains any rows|
|NOT EXISTS()|	Whether the result of a query contains no rows|
|IN()|	Whether a value is within a set of values|
|NOT IN()|	Whether a value is not within a set of values|

<U>Example</U>: `EXISTS()`, `NOT EXISTS()`

| col  |
|------|
| aaa  |
| bbb  |
| ccc  |
| eee  |

---

```sql
SELECT EXISTS
    (SELECT *
    FROM t
    WHERE col LIKE 'c%');
-- start with 'c'
-- EXIST? -> TRUE
-- 1
```
```sql
SELECT NOT EXISTS
    (SELECT *
    FROM t
    WHERE col LIKE 'c%');
-- start with 'c'
-- NOT EXIST? -> FALSE
-- 1
```

<U>Example</U>: `IN()`, `NOT IN()`

```sql
SELECT 2 IN (0, 3, 5, 7);
-- FALSE -> 0

SELECT 2 NOT IN (0, 3, 5, 7);
-- TRUE -> 1
```
```sql
SELECT 'wefwf' IN ('wee', 'wefwf', 'weg');
-- TRUE -> 1

SELECT 'wefwf' NOT IN ('wee', 'wefwf', 'weg');
-- FALSE -> 0
```
```SQL
[TYPE CAUTION]
-- wrong query
SELECT val1 FROM tbl1 WHERE val1 IN (1, 2, 'a');

-- correct query
SELECT val1 FROM tbl1 WHERE val1 IN ('1', '2', 'a');

SELECT 'a' IN (0);
-- 'a'는 int 타입으로 변형 시 0
-- TRUE -> 1

SELECT 0 IN ('b');
-- 'b'는 int 타입으로 변형 시 0
-- TRUE -> 1
```
```sql
[NULL IN IN()]

SELECT NULL IN (1, 2, 3);
-- NULL

SELECT 1 IN (2, 3, NULL);
-- NULL
```

### 3. Special Comparison Operators

|Name|	Description|
|--|--|
|IS|	Test a value against a boolean|
|IS NOT|	Test a value against a boolean|
|IS NOT NULL|	NOT NULL value test|
|IS NULL|	NULL value test|
|ISNULL()|	Test whether the argument is NULL|
|STRCMP()|	Compare two strings|

```SQL
[IS()]
SELECT
    1 IS TRUE, -- 1
    0 IS FALSE, -- 1
    NULL IS UNKNOWN; -- 1

[IS NOT()]
SELECT
    1 IS NOT UNKNOWN, -- 1 
    0 IS NOT UNKNOWN, -- 1
    NULL IS NOT UNKNOWN; -- 0
```

```SQL
[IS NULL()]
SELECT
    1 IS NULL, -- 0
    0 IS NULL, -- 0
    NULL IS NULL; -- 1

[IS NOT NULL()]
SELECT
    1 IS NOT NULL, -- 1
    0 IS NOT NULL, -- 1
    NULL IS NOT NULL; -- 0
```

```SQL
[ISNULL(expr)]
SELECT ISNULL(1+1); -- 0
SELECT ISNULL(1/0); -- INFINITE == NULL
                    -- 1
```

### 4. Others

|Name|	Description|
|--|--|
|COALESCE()|	Return the first non-NULL argument|
|INTERVAL()|	Return the index of the argument that is less than the first argument|

1. `COALESCE(a, b, c, ...)`: Returns the first non-NULL value
```SQL
SELECT COALESCE(NULL, 1, 2); -- 1

SELECT COALESCE(NULL, NULL, NULL); -- NULL

SELECT COALESCE(NULL, 'hello', 'world'); -- 'hello'
```

2. `INTERVAL(N, N1, N2, ...)`: Returns the index of the interval in which N fits (uses binary search, very fast)
```SQL
SELECT INTERVAL(23, 1, 15, 17, 30, 44, 200);
-- 23은 4번째이므로 index 3 반환

SELECT INTERVAL(10, 1, 10, 100, 1000);
-- 10은 2번째이지만
-- ≤ N2 조건을 만족하면 index+1을 반환하므로 index 2 반환

SELECT INTERVAL(22, 23, 30, 44, 200);
-- 22는 첫번째이므로 index 0 반환

SELECT INTERVAL(NULL, 1, 10, 100);
-- N이 NULL이면 비교 자체가 불가능하므로 항상 -1 반환
```

# 문제 풀이하기

## 문제1: Type of Triangle
> CASE WHEN, GREATEST()

### 요구사항
Write a query identifying the type of each record in the TRIANGLES table using its three side lengths. Output one of the following statements for each record in the table:
- Equilateral: It's a triangle with  sides of equal length.
- Isosceles: It's a triangle with  sides of equal length.
- Scalene: It's a triangle with  sides of differing lengths.
- Not A Triangle: The given values of A, B, and C don't form a triangle.

### 작성한 쿼리
```sql
SELECT
    CASE
        WHEN GREATEST(A, B, C) >= (A + B + C - GREATEST(A, B, C)) THEN 'Not A Triangle'
        WHEN A=B AND B=C THEN 'Equilateral'
        WHEN A=B OR B=C OR C=A THEN 'Isosceles'
        ELSE 'Scalene' END
FROM TRIANGLES
```

## 문제2: Find Customer Referee
> IS NULL, <>

### 요구사항
Write a query identifying the type of each record Find the names of the customer that are not referred by the customer with id = 2.

Return the result table in any order.

The result format is in the following example.

### 작성한 쿼리
```sql
SELECT NAME
FROM CUSTOMER
WHERE REFEREE_ID IS NULL OR REFEREE_ID <> 2
```