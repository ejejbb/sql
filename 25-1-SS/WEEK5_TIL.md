# Week5_TIL

## 14.8.2 Regular Expressions

### 1. Key Regular Expression Functions / Operators

| Function / Operator | Description |
|---------------------|-------------|
| `expr REGEXP pat` / `RLIKE` | Returns 1 if the string matches the pattern |
| `expr NOT REGEXP pat` | Negation of the above |
| `REGEXP_LIKE(expr, pat[, match_type])` | ICU-based comparison (MySQL 8.0+) |
| `REGEXP_INSTR(expr, pat, ...)` | Returns the starting index of the match |
| `REGEXP_SUBSTR(expr, pat, ...)` | Returns the matched substring |
| `REGEXP_REPLACE(expr, pat, repl, ...)` | Replaces matched pattern with another string |

### 2. Regular Expression Pattern Syntax

| Pattern | Meaning |
|--------|---------|
| `^` / `$` | Start / End of string |
| `.` | Any character (excluding newline by default) |
| `*` / `+` / `?` | 0 or more / 1 or more / 0 or 1 times |
| `a{3}` / `a{1,3}` | Exactly 3 times / between 1 and 3 times |
| `[abc]` / `[^abc]` | One of a, b, c / None of a, b, c |
| `[a-z]` | Character range |
| `|` | OR condition (`abc|def`) |
| `(abc)*` | Repeat group |
| `[:digit:]` | Digit class (recommend using `[0-9]` for performance) |

### 3. `expr (NOT) REGEXP pat` / `RLIKE` Examples

```SQL
SELECT 'apple' REGEXP '^a';     -- 1 (starts with a)
SELECT 'banana' REGEXP '^a';    -- 0 (does not start with a)

SELECT 'carrot' RLIKE 'r';      -- 1

SELECT 'test' NOT REGEXP 't$';  -- 0 (ends with t)
SELECT 'test' NOT REGEXP 'z';   -- 1 (no z)
```

### 4. `REGEXP_LIKE()` Examples
> `REGEXP_LIKE(expr, pat[, match_type])`

```sql
-- Case-insensitive
SELECT REGEXP_LIKE('abc', 'ABC'); -- 1

-- Case-sensitive(대소문자 구분O)
SELECT REGEXP_LIKE('abc', 'ABC', 'c'); -- 0

-- Pattern beginning with B, one or more a's, ending in n
SELECT REGEXP_LIKE('Banana', '^Ba+n'); -- 1
SELECT REGEXP_LIKE('Banana', '^Ba+n$');   -- 0
```

**`match_type` options:**
- `'c'`: Case-sensitive 대소문자 구분O
- `'i'`: Case-insensitive (default) 대소문자 구분X
- `'m'`: Multiline mode (match `.` with newlines)

### 5. `REGEXP_INSTR()` Examples
> Returns the **starting index** of the match, including spaces.

```sql
SELECT REGEXP_INSTR('dog cat dog', 'dog');        -- 1
SELECT REGEXP_INSTR('dog cat dog', 'dog', 2);     -- 9
SELECT REGEXP_INSTR('aa aaa aaaa', 'a{4}');       -- 8

SELECT REGEXP_INSTR('dog cat dog', 'dog', 1, 1, 0);  -- 1
SELECT REGEXP_INSTR('dog cat dog', 'dog', 1, 1, 1);  -- 4
```

Optional arguments: `REGEXP_INSTR(expr, pat, pos, occurrence, return_option)`
- `pos`: Start position (default: 1)
- `occurrence`: Match count to find
- `return_option`: 0 = start index, 1 = end index + 1


### 6. `REGEXP_REPLACE()` Examples

```sql
SELECT REGEXP_REPLACE('a b c', 'b', 'X'); -- a X c
SELECT REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X', 1, 3); -- abc def X
SELECT REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X'); -- X X X
```

### 7. `REGEXP_SUBSTR()` Examples
> Returns the matched substring

```sql
SELECT REGEXP_SUBSTR('abc def ghi', '[a-z]+');       -- abc
SELECT REGEXP_SUBSTR('abc def ghi', '[a-z]+', 1, 3); -- ghi
```

## 14.12 Bit Functions and Operators

### 1. Bitwise Operators
| Operator | Name        | Description                           |                                |
| -------- | ----------- | ------------------------------------- | ------------------------------ |
| `&`      | AND         | 1 only if both bits are 1   |                                |
|\|         | OR                     | 1 if either bit is 1 |
| `^`      | XOR         | 1 if the bits are different |                                |
| `~`      | NOT         | Inverts each bit                      |                          
| `<<`     | Left Shift  | Shifts bits to the left               |                               
| `>>`     | Right Shift | Shifts bits to the right              |                               

**Example:**
| Expression | Explanation               | Result |
| ---------- | ------------------------- | ------ |
| `6 & 3`    | 110 & 011 = 010           | 2      |
| `6 \| 3`   | 110 \| 011 = 111          | 7      |
| `6 ^ 3`    | 110 ^ 011 = 101           | 5      |
| `~6`       | Inverts bits (binary NOT)<br> 110 → 001 (x)<br> 컴퓨터는 64비트 기준으로 계산하므로<br> ~6 = 11...11111001<br> 2의 보수법에서 `~x == -x - 1` | -7     |
| `6 << 1`   | 00000110 → 00001100<br> `x << n == x × 2ⁿ`       | 12     |
| `6 >> 1`   | 00000110 → 00000011<br> `x << n == x ÷ 2ⁿ`       | 3      |


### 2. Bitwise Aggregate Functions

| Function        | Description                         |
| --------------- | ----------------------------------- |
| `BIT_COUNT(N)`  | Returns the number of 1 bits in N   |
| `BIT_AND(expr)` | Performs bitwise AND across a group |
| `BIT_OR(expr)`  | Performs bitwise OR across a group  |
| `BIT_XOR(expr)` | Performs bitwise XOR across a group |

