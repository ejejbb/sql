# Week6_TIL
> **문제 풀이하기**

## 문제1: 복수 국적 메달 수상한 선수 찾기

### 요구사항
2000년 이후의 메달 수상 기록만 고려했을 때, 메달을 수상한 올림픽 참가 선수 중 2개 이상의 국적으로 메달을 수상한 기록이 있는 선수의 이름을 조회하는 쿼리를 작성해주세요. 조회된 선수의 이름은 오름차순으로 정렬되어 있어야 합니다.

### 작성한 쿼리

```
- athletes 테이블: 역대 올림픽 참가 선수의 **이름**, 선수id
- games 테이블: 올림픽 *개최 연도*, 개최 도시와 시즌 정보, 게임id
- records 테이블: 역대 올림픽 참가 선수들의 신체 정보와 획득한 *메달 정보*, 다른 테이블과 매핑할 수 있는 ID 정보, 경기id, 기록id, 선수id, 게임id, 팀id
- teams 테이블: *국가 정보*, 팀id
→ team 테이블을 확인해보니 팀id별로 국가가 하나씩 대응되는 것을 확인할 수 있었음. 따라서 team 테이블을 사용하지 않고 records 테이블의 팀id로 국가 구분 가능

2000년 이후의 메달 수상 기록만 고려
메달을 수상한 올림픽 참가 선수 중
2개 이상의 국적으로 메달을 수상한 기록이 있는 선수의 이름을 조회
조회된 선수의 이름은 오름차순으로 정렬
```

```sql
SELECT name
FROM athletes AS A
JOIN records AS R
    ON A.id = R.athlete_id
JOIN games AS G
    ON R.game_id = G.id
WHERE G.year >= 2000
    AND R.medal IS NOT NULL
GROUP BY R.athlete_id
HAVING count(DISTINCT R.team_id) >= 2
ORDER BY name;
```

## 문제2: 온라인 쇼핑몰의 월 별 매출액 집계

### 요구사항
이 온라인 쇼핑몰의 월 별 매출 규모를 한 눈에 파악할 수 있는 데이터를 만들고 싶습니다. 위 두 테이블의 데이터를 조합해 월 별로 취소 주문을 제외한 주문 금액의 합계, 취소 주문의 금액 합계, 그리고 총 합계를 계산하는 쿼리를 작성해주세요. order_id가 C로 시작하는 주문이 취소 주문입니다. 결과 데이터는 아래 4개 컬럼을 포함해야 하고 order_month 컬럼의 값으로 오름차순 정렬되어 있어야 합니다.

### 작성한 쿼리
```sql
SELECT
  substr(order_date, 1, 7) order_month,
  sum(CASE WHEN i.order_id NOT LIKE 'C%' THEN price * quantity ELSE 0 END) ordered_amount,
  sum(CASE WHEN i.order_id LIKE 'C%' THEN price * quantity ELSE 0 END) canceled_amount,
  sum(price * quantity) total_amount
FROM order_items i
LEFT JOIN orders o ON i.order_id = o.order_id
GROUP BY 1
ORDER BY 1;
```

## 문제3: 세 명이 서로 친구인 관계 찾기

### 요구사항
주어진 데이터를 활용해 ID가 3820인 사용자를 포함해 세 명의 사용자가 친구 관계인 경우를 모두 출력하는 쿼리를 작성해주세요. 쿼리 결과에는 아래 컬럼이 포함되어 있어야 합니다.

### 작성한 쿼리
```sql
SELECT
  A.user_a_id "user_a_id",
  B.user_a_id "user_b_id",
  B.user_b_id "user_c_id"
FROM
  edges A
  JOIN edges B ON A.user_b_id = B.user_a_id
  JOIN edges C ON A.user_a_id = C.user_a_id AND B.user_b_id = C.user_b_id
WHERE
  A.user_a_id = 3820
  OR B.user_a_id = 3820
  OR B.user_b_id = 3820;
```