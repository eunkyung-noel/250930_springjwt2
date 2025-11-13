# 01*DQL*기본\_SELECT

#DQL #SELECT #데이터조회 #쿼리 #query

---

## 🎯 학습 목표

- `SELECT` 문의 기본 구조와 주요 절(Clause)의 역할을 이해합니다.
- `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`를 사용하여 데이터를 필터링, 그룹화, 정렬, 제한하는 방법을 익힙니다.

---

### `SELECT` 기본 구조

```sql
SELECT [DISTINCT] column_list
FROM   table_name
[WHERE  filter_conditions]
[GROUP BY column_list]
[HAVING group_filter_conditions]
[ORDER BY column_list [ASC|DESC]]
[LIMIT  row_count [OFFSET offset]];
```

#### SQL 실행 순서 (논리적)

> `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`

#실행순서 #executionorder

---

### 1. 기본 조회 (`SELECT`, `FROM`, `DISTINCT`)

- **모든 컬럼 조회**: `SELECT * FROM customers;`
- **특정 컬럼만 조회**: `SELECT name, email FROM customers;`
- **중복 제거 조회**: `SELECT DISTINCT city FROM customers;`

---

### 2. 조건 필터링 (`WHERE`)

- **비교 연산자**: `age <= 30`, `city = '서울'`
- **논리 연산자**: `AND`, `OR`
  ```sql
  -- 25세 이상이고 서울에 사는 고객
  SELECT * FROM customers WHERE age >= 25 AND city = '서울';
  ```
- **범위 조건**: `BETWEEN`
  ```sql
  -- 25세에서 30세 사이의 고객
  SELECT * FROM customers WHERE age BETWEEN 25 AND 30;
  ```
- **패턴 매칭**: `LIKE`
  ```sql
  -- 이름이 '수'로 끝나는 고객
  SELECT name FROM customers WHERE name LIKE '%수';
  ```
- **목록 포함 여부**: `IN`
  ```sql
  -- 서울 또는 인천에 사는 고객
  SELECT * FROM customers WHERE city IN ('서울', '인천');
  ```

#WHERE #필터링 #filtering #연산자 #operator

---

### 3. 그룹화 및 집계 (`GROUP BY`, `HAVING`, 집계 함수)

- **`GROUP BY`**: 특정 컬럼을 기준으로 데이터를 그룹화합니다.
- **집계 함수**: `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()` 등 그룹별 계산을 수행합니다.
- **`HAVING`**: `GROUP BY`로 생성된 그룹에 대한 조건을 지정합니다. (`WHERE`는 그룹화 전 개별 행에 대한 조건)

```sql
-- 도시별 고객 수
SELECT city, COUNT(*) AS customer_count
FROM customers
GROUP BY city;

-- 고객 수가 2명 이상인 도시만 조회
SELECT city, COUNT(*) AS customer_count
FROM customers
GROUP BY city
HAVING COUNT(*) >= 2;

-- 고객별 총 주문 금액
SELECT customer_id, SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id;
```

#GROUPBY #그룹화 #HAVING #집계함수 #aggregatefunction

---

### 4. 정렬 및 제한 (`ORDER BY`, `LIMIT`)

- **`ORDER BY`**: 결과를 특정 컬럼 기준으로 정렬합니다. (`ASC`: 오름차순, `DESC`: 내림차순)
- **`LIMIT`**: 반환할 행의 수를 제한합니다. 페이징 구현에 사용됩니다.

```sql
-- 나이순으로 고객 정렬 (내림차순)
SELECT * FROM customers ORDER BY age DESC;

-- 가격이 가장 비싼 상품 2개 조회
SELECT * FROM products ORDER BY price DESC LIMIT 2;

-- 2페이지 내용 조회 (페이지당 3개)
SELECT * FROM customers ORDER BY customer_id LIMIT 3 OFFSET 3;
-- OFFSET 3: 3개의 행을 건너뛰고, LIMIT 3: 3개의 행을 가져옴
```

#ORDERBY #정렬 #sorting #LIMIT #페이징 #paging

---

### 통합 연습 (JOIN 맛보기)

```sql
-- 최근 30일간 매출 TOP 5 고객 조회
SELECT
    c.name,
    c.email,
    SUM(o.total_amount) AS total_spent
FROM orders AS o
JOIN customers AS c ON o.customer_id = c.customer_id
WHERE o.order_date >= CURDATE() - INTERVAL 30 DAY
GROUP BY c.customer_id
ORDER BY total_spent DESC
LIMIT 5;
```

- **`JOIN`**: 두 개 이상의 테이블을 연결합니다.
- **`AS` (Alias)**: 테이블이나 컬럼에 별칭을 부여하여 쿼리를 간결하게 만듭니다.

#JOIN #조인 #Alias #별칭
