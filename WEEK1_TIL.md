# Week1_TIL

## Window Function

<u>Nonaggregate window functions</u> that, for each row from a query, perform a calculation using rows related to that row.

<u>Aggregate functions</u> also can be used as window functions using *OVER clause*.


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


### *Over* clause _ Nonaggregate window function

1. ROW_NUMBER(): 중복 없이 순번 부여    
- rows numbers range from 1 to the number of partition rows
- `ORDER BY` affects the order in which rows are numbered
- assigns peers different row numbers

2. RANK(): 중복 순번 부여, 다음 순번 건너뜀
- 