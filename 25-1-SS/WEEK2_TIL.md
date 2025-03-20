# Week2_TIL

## 15.2.13.2 JOIN Clause

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

-- can reference multiple tables
-- FROM table1, table2
-- but using JOIN is recommendable

```SQL
table_factor: {
    tbl_name [PARTITION (partition_names)] -- 특정 파티션만 조회
        [[AS] alias] [index_hint_list]
  | [LATERAL] table_subquery [AS] alias [(col_list)] -- 서브쿼리에서 외부 테이블 값 참조 가능
  | ( table_references )
}
```

-- 외부쿼리 참조 예시
-- SELECT * FROM customers c,
-- LATERAL (SELECT * FROM orders o WHERE o.customer_id = c.id) AS sub;

```sql
joined_table: {
    table_reference {[INNER | CROSS] JOIN | STRAIGHT_JOIN} table_factor [join_specification]
  | table_reference {LEFT|RIGHT} [OUTER] JOIN table_reference join_specification
  | table_reference NATURAL [INNER | {LEFT|RIGHT} [OUTER]] JOIN table_factor
}
-- INNER JOIN = JOIN
-- STRAIGHT_JOIN is similar to JOIN, except that the left table is always read before the right table.
-- LEFT/RIGHT JOIN: OUTER JOIN, 

join_specification: {
    ON search_condition
  | USING (join_column_list)
}

join_column_list:
    column_name[, column_name] ...

index_hint_list:
    index_hint[ index_hint] ...

index_hint: {
    USE {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] ([index_list])
  | {IGNORE|FORCE} {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] (index_list)
}

index_list:
    index_name [, index_name] ...
```