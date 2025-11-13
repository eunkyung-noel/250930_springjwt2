# 02*DML*데이터\_조작

#DML #데이터조작 #INSERT #UPDATE #DELETE #트랜잭션

---

## 🎯 학습 목표

- `INSERT`, `UPDATE`, `DELETE`를 사용하여 데이터를 추가, 수정, 삭제하는 방법을 익힙니다.
- 데이터 조작 시 발생할 수 있는 위험을 이해하고, 트랜잭션의 기본 개념을 파악합니다.

---

### 1. 데이터 삽입 (`INSERT`)

```sql
-- 기본 구문
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);

-- 예시: 새로운 고객 추가
INSERT INTO customers (name, email, age, city)
VALUES ('홍길동', 'hong@email.com', 29, '광주');

-- 여러 행 한 번에 추가
INSERT INTO products (name, category, price, stock_quantity) VALUES
('LG 그램', '전자제품', 1800000.00, 15),
('삼성 오디세이', '전자제품', 2200000.00, 10);
```

- **`UPSERT`**: 데이터가 없으면 `INSERT`, 있으면 `UPDATE`를 수행하는 기능입니다. (MySQL: `ON DUPLICATE KEY UPDATE ...`)
- **`DEFAULT` 값 활용**: 테이블 정의 시 `DEFAULT` 값을 설정해두면 `INSERT` 시 해당 컬럼을 생략할 수 있어 코드가 간결해지고 데이터 무결성을 확보할 수 있습니다.

#INSERT #데이터삽입 #UPSERT #업서트

---

### 2. 데이터 수정 (`UPDATE`)

```sql
-- 기본 구문
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;

-- 예시: 특정 고객의 도시 변경
UPDATE customers
SET city = '제주'
WHERE email = 'kim@email.com';

-- 예시: '의류' 카테고리 상품 전체 10% 할인
UPDATE products
SET price = price * 0.9
WHERE category = '의류';
```

> **⚠️ 치명적인 실수: `WHERE` 절 생략** > `UPDATE` 문에서 `WHERE` 절을 생략하면 **테이블의 모든 행**이 수정됩니다. 이는 데이터 손실로 이어질 수 있는 매우 위험한 작업입니다.
> **예방책**:
>
> 1.  `UPDATE` 실행 전, 동일한 `WHERE` 조건으로 `SELECT`를 먼저 실행하여 변경 대상을 확인합니다.
> 2.  MySQL Workbench 등의 클라이언트에서 **Safe Updates** 옵션을 활성화합니다.

#UPDATE #데이터수정 #WHERE #실수사례

---

### 3. 데이터 삭제 (`DELETE`)

```sql
-- 기본 구문
DELETE FROM table_name
WHERE condition;

-- 예시: 특정 주문 기록 삭제
DELETE FROM orders
WHERE order_id = 1;
```

> **⚠️ 치명적인 실수: `WHERE` 절 생략** > `DELETE` 문 역시 `WHERE` 절을 생략하면 **테이블의 모든 데이터가 삭제**됩니다.
> **고려사항**:
>
> - **Soft Delete**: 실제 데이터를 삭제하는 대신, `is_deleted` 같은 컬럼을 두어 `true`로 `UPDATE`하는 방식입니다. 데이터 복구가 용이하여 많은 서비스에서 채택합니다.
> - **대량 삭제**: 수백만 건 이상의 데이터를 삭제할 때는 서버 부하와 잠금(Lock) 시간을 줄이기 위해 `LIMIT`를 사용하여 여러 번에 나누어 실행하는 것이 안전합니다. (예: `DELETE FROM logs WHERE ... LIMIT 5000;`)

#DELETE #데이터삭제 #SoftDelete #소프트딜리트

---

### 트랜잭션 (Transaction) 한 줄 정리

- **정의**: '모두 성공하거나 모두 실패해야 하는' 논리적인 작업 단위.
- **주요 명령어**: `START TRANSACTION`, `COMMIT` (변경사항 영구 저장), `ROLLBACK` (변경사항 취소).
- **기본 설정**: 대부분의 최신 RDBMS는 **autocommit = ON**이 기본입니다. 이는 모든 DML 쿼리가 실행 즉시 `COMMIT`됨을 의미합니다.
- **사용 시점**: 여러 DML(예: `UPDATE` 후 `INSERT`)을 하나의 원자적(atomic) 작업으로 묶고 싶을 때만 명시적으로 `START TRANSACTION`을 사용합니다.

```sql
START TRANSACTION;

-- 1. 재고 감소
UPDATE products SET stock_quantity = stock_quantity - 1 WHERE product_id = 1;
-- 2. 주문 기록 추가
INSERT INTO orders (customer_id, product_id, ...) VALUES (1, 1, ...);

COMMIT; -- 모든 작업이 성공하면 확정
-- 만약 중간에 오류가 발생하면 ROLLBACK;
```

#트랜잭션 #transaction #COMMIT #커밋 #ROLLBACK #롤백 #autocommit
