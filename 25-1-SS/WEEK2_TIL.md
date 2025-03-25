# Week2_TIL

## 15.2.13.2 JOIN Clause

### 1. table reference
```sql
table_references:
    escaped_table_reference [, escaped_table_reference] ...
```
A table reference is also known as a join expression. It may contain a `PARTITION` clause.

**💡 PARTITION Clause**   
By adding the `PARTITION` clause right after the table name, you can limit the query to access only the specified partitions.

<U>EXAMPLE.</U> WHERE vs. PARTITION
```SQL
-- WHERE
SELECT * FROM sales WHERE year = 2023;
```
The database scans the entire table, evaluates the `WHERE` condition, and returns the matching rows.
```sql
-- PARTITION
CREATE TABLE sales (
  id INT,
  year INT,
  amount DECIMAL(10,2)
)
PARTITION BY RANGE (year) (
  PARTITION p2022 VALUES LESS THAN (2023),
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025)
);
```
The data is physically divided into three partitions: `p2022`, `p2023`, and `p2024`. When you run the query `WHERE year = 2023`, MySQL automatically knows from the partition metadata that it only needs to access p2023. Other partitions are completely ignored. This optimization is called **Partition Pruning**.

```SQL
table_factor: {
    tbl_name [PARTITION (partition_names)]
        [[AS] alias] [index_hint_list]
  | [LATERAL] table_subquery [AS] alias [(col_list)] -- 서브쿼리에서 외부 테이블 값 참조 가능
  | ( table_references )
}
```

### 2. `JOIN`: multiple table
In standard SQL, you can only place a **single table** inside parentheses. However, MYSQL allows multiple table inside parentheses.
```sql
SELECT * FROM t1 LEFT JOIN (t2, t3, t4)
ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c);
```
The comma behaves like a **CROSS JOIN** in MYSQL.
```
(t2, t3, t4)
== (t2 CROSS JOIN t3 CROSS JOIN t4)
```

### 3. Various kinds of `JOIN`
```sql
joined_table: {
    table_reference {[INNER | CROSS] JOIN | STRAIGHT_JOIN} table_factor [join_specification]
  | table_reference {LEFT|RIGHT} [OUTER] JOIN table_reference join_specification
  | table_reference NATURAL [INNER | {LEFT|RIGHT} [OUTER]] JOIN table_factor
}
```
1. INNER JOIN = JOIN
2. STRAIGHT_JOIN
- similar to JOIN, except that the left table is always read before the right table.
- ensures the order of joins   
  <u>example</u>
  ```sql
  SELECT * FROM t1 STRAIGHT_JOIN t2 ON t1.id = t2.id;
  -- join order: t1 -> t2
  ```
3. LEFT/RIGHT JOIN
- OUTER JOIN
- If there is no matching row for the right table in the ON or USING part in a LEFT JOIN, a row with all columns set to NULL is used for the right table.
4. NATURAL_JOIN
- automatically joins two tables based on columns with the same name
- No need to specify `ON` or `USING` clauses.
```
[procedure]

#1. coalesced common columns of the two joined tables, in the order in which they occur in the first table

#2. columns unique to the first table

#3. columns unique to the second table
```
**❓COALESCE**
```SQL
COALESCE(x, y) = (CASE WHEN x IS NOT NULL THEN x ELSE y END)
```

### 4. Precedence: `JOIN` vs. comma operator
JOIN has higher precedence than the comma operator (,)
```SQL
[⚠️wrong clause]
SELECT * FROM t1, t2 JOIN t3 ON t1.id = t3.id;
```
```SQL
[correct clause]
SELECT * FROM (t1, t2) JOIN t3 ON t1.id = t3.id;
-- OR
SELECT * FROM t1 JOIN t2 JOIN t3 ON t1.id = t3.id;
```

### 5. `USING` and `ON`

```sql
join_specification: {
    ON search_condition
  | USING (join_column_list)
}
```

| Item              | USING                                          | ON                                          |
|-------------------|------------------------------------------------|---------------------------------------------|
| Condition Syntax  | Automatically detects common columns           | Requires explicit join condition            |
| `SELECT *` Result | Outputs a single column via `COALESCE(a.col, b.col)` | Outputs both `a.col` and `b.col`            |
| Readability       | Simple and clear                               | Allows for complex join conditions          |


## 14.19.3 MySQL Handling of `GROUP BY`

### 1. Differences from Standard SQL
- **SQL-92**:  
  Does **not allow non-aggregated columns** in `SELECT`, `HAVING`, or `ORDER BY` unless they appear in the `GROUP BY` clause.  
  ```sql
  [⚠️wrong clause(invalid in SQL-92)]
  SELECT o.custid, c.name, MAX(o.payment)
  FROM orders o, customers c
  WHERE o.custid = c.custid
  GROUP BY o.custid;
  -- `c.name` is not in GROUP BY → illegal
  ```
- **SQL:1999 and later**:  
  Allows non-aggregated columns if they are **functionally dependent** on the `GROUP BY` columns.  
  e.g., If `custid` is a **PK**, then `c.name` is allowed.

### 2. <u>Aggregate Query</u> without `GROUP BY`
```sql
[⚠️wrong clause]
SELECT name, MAX(age) FROM t;
-- `name` is not aggregated or grouped
```
```sql
[correct clause]
-- Use `ANY_VALUE(name)` to resolve the error
-- show any name among those with the maximum age
SELECT ANY_VALUE(name), MAX(age) FROM people;
```

### 3. Aliases in `HAVING`, `GROUP BY`(allowed)
```sql
-- HAVING
SELECT
  name,
  COUNT(*) AS c
FROM orders
GROUP BY name
HAVING c = 1;
-- Allowed in MySQL, not in standard SQL
-- standard SQL => HAVING COUNT(*) = 1;
```
```sql
-- GROUP BY
SELECT id, FLOOR(value/100) AS val
FROM tbl_name
GROUP BY id, val;
-- Allowed in MySQL, not in standard SQL
-- standard SQL => GROUP BY id, FLOOR(value/100);
```

### 4. Limitations of Functional Dependency Detection on Expressions

```sql
SELECT id, FLOOR(value/100), id + FLOOR(value/100)
FROM tbl_name
GROUP BY id, FLOOR(value/100);
-- `id + FLOOR(...)` is not recognized as functionally dependent
```
```sql
-- SOLUTION: subquery
SELECT id, F, id + F
FROM (
  SELECT id, FLOOR(value/100) AS F
  FROM tbl_name
  GROUP BY id, FLOOR(value/100)
) AS dt;
```

## 15.2.13 SELECT Statement - `HAVING`

### 1. HAVING vs WHERE

| Clause  | WHERE                                  | HAVING                                 |
|---------|----------------------------------------|-----------------------------------------|
| Timing  | - Applied **before** `GROUP BY`<br> - filters individual rows | - Applied **after** `GROUP BY`<br> - filters groups |
| Use of Aggregate function | X     |O           |
| Use     | Filters rows                      | Filters aggregated results (groups)     |

<u>Example:</u> Wrong use of `HAVING`, `WHERE`
```sql
[⚠️wrong clause]
SELECT col_name
FROM tbl_name
HAVING col_name > 0;
-- HAVING col_name > 0; -> WHERE col_name > 0;
-- `WHERE` filters individual rows
-- `HAVING` filters groups after aggregation
-- col_name is an individual row
```
```sql
[⚠️wrong clause]
SELECT user, MAX(salary)
FROM users
GROUP BY user
WHERE MAX(salary) > 10;
-- WHERE MAX(salary) > 10; -> HAVING MAX(salary) > 10;
-- use `HAVING` when filtering aggregate results like `MAX()`, `SUM()`, `COUNT()` etc
```

### 2. Execution Order of `HAVING`
![image6](/image/image6.png)

### 3. Differences from Standard SQL

#### Standard SQL:
- The `HAVING` clause can reference only:
  - Columns that appear in the `GROUP BY` clause
  - Aggregate functions

#### MySQL Extensions:
- In MySQL, `HAVING` can reference:
  - Aliases defined in the `SELECT` list
  - Columns from outer subqueries
- ⚠️ **Ambiguous references** may generate a **warning**.

<u>Example (ambiguous reference):</u>
```sql
SELECT COUNT(col1) AS col2
FROM t
GROUP BY col2
HAVING col2 = 2;
-- it's unclear whether `col2` in `HAVING` refers to the alias in `SELECT` or the grouped column
-- MySQL prefers `GROUP BY` column
```