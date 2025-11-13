# 03*DDL*데이터\_정의

#DDL #데이터정의 #CREATE #ALTER #DROP #TRUNCATE #데이터타입

---

## 🎯 학습 목표

- `CREATE TABLE`을 사용하여 테이블 스키마를 정의하고, 주요 제약 조건과 데이터 타입을 이해합니다.
- `ALTER TABLE`로 기존 테이블의 구조를 변경하는 방법을 익힙니다.
- `DROP`과 `TRUNCATE`의 차이점을 파악하고, 상황에 맞게 사용합니다.

---

### 1. 테이블 생성 (`CREATE TABLE`)

`CREATE TABLE`은 데이터베이스의 '설계도'를 그리는 작업입니다. 컬럼 이름, 데이터 타입, 제약 조건을 정의합니다.

```sql
CREATE TABLE employees (
  id         INT PRIMARY KEY AUTO_INCREMENT,
  name       VARCHAR(50)   NOT NULL,
  dept       VARCHAR(20)   DEFAULT 'General',
  salary     INT           CHECK (salary >= 0),
  hired_at   DATETIME      DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT uk_name_dept UNIQUE (name, dept)
);
```

#### 제약 조건 (Constraints)

- **`PRIMARY KEY` (PK)**: 각 행을 **고유하게 식별**하는 키. `NOT NULL`과 `UNIQUE` 속성을 모두 가집니다.
- **`FOREIGN KEY` (FK)**: 다른 테이블의 PK/UK를 **참조**하여 테이블 간의 관계를 맺고, 데이터 무결성을 보장합니다.
- **`UNIQUE`**: 컬럼 내의 모든 값이 중복되지 않도록 보장합니다. (단, `NULL`은 한 번 허용될 수 있음)
- **`NOT NULL`**: 해당 컬럼에 `NULL` 값이 들어갈 수 없도록 강제합니다.
- **`DEFAULT`**: 값이 명시적으로 주어지지 않았을 때 사용될 기본값을 지정합니다.
- **`CHECK`**: 컬럼에 들어갈 수 있는 값의 조건을 지정합니다.

#제약조건 #constraints #PK #FK #UNIQUE #NOTNULL #DEFAULT #CHECK

---

### 주요 데이터 타입

| 분류          | 대표 타입                           | 설명 및 사용 사례                                                               |
| :------------ | :---------------------------------- | :------------------------------------------------------------------------------ |
| **정수형**    | `TINYINT`, `INT`, `BIGINT`          | 각각 1, 4, 8바이트. `AUTO_INCREMENT` PK에 주로 `INT` 또는 `BIGINT` 사용.        |
| **소수형**    | `DECIMAL(p, s)`                     | **고정 소수점**. 금액 등 정확한 계산이 필요할 때 사용. (예: `DECIMAL(10, 2)`)   |
| **실수형**    | `FLOAT`, `DOUBLE`                   | **부동 소수점**. 근사값으로, 미세한 오차가 발생할 수 있어 금융 계산에는 부적합. |
| **문자열**    | `CHAR(n)`, `VARCHAR(n)`, `TEXT`     | `CHAR`(고정 길이), `VARCHAR`(가변 길이). 65,535 바이트 초과 시 `TEXT` 사용.     |
| **날짜/시간** | `DATE`, `DATETIME`, `TIMESTAMP`     | `DATE`(날짜), `DATETIME`(시간), `TIMESTAMP`(UTC 자동 변환 및 타임존 처리).      |
| **불린형**    | `BOOLEAN` (내부적으로 `TINYINT(1)`) | `TRUE`(1), `FALSE`(0)를 저장.                                                   |
| **열거형**    | `ENUM('A', 'B', ...)`               | 미리 정의된 문자열 값 중 하나만 저장. 값 추가 시 `ALTER TABLE` 필요.            |
| **JSON**      | `JSON`                              | 반정형 데이터를 저장. 특정 키 값으로 조회하거나 인덱싱 가능.                    |

> **💡 타입 선택 가이드**
>
> - **저장 공간 vs 성능 vs 정밀도** 사이의 균형을 고려해야 합니다.
> - 문자열은 예상 최대 길이보다 약간 여유를 두어 `VARCHAR` 길이를 지정하는 것이 일반적입니다.
> - 날짜/시간은 **UTC**로 저장하고, 애플리케이션 단에서 사용자의 지역 시간으로 변환하여 보여주는 것을 권장합니다.

#데이터타입 #datatype #정수 #문자열 #날짜 #UTC

---

### 2. 테이블 구조 변경 (`ALTER TABLE`)

이미 생성된 테이블의 구조를 변경합니다.

```sql
-- 컬럼 추가
ALTER TABLE employees ADD COLUMN phone VARCHAR(20) AFTER name;

-- 컬럼 타입 변경
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(11, 2);

-- 인덱스 추가 (SELECT 성능 향상)
ALTER TABLE orders ADD INDEX idx_order_date (order_date);

-- 컬럼 이름 변경
ALTER TABLE employees RENAME COLUMN name TO employee_name;

-- 컬럼 삭제
ALTER TABLE employees DROP COLUMN phone;
```

> **⚠️ 운영 중인 DB에서의 `ALTER TABLE`**
> 대용량 테이블에 `ALTER TABLE`을 실행하면 테이블 전체에 잠금(Lock)이 걸려 서비스 장애를 유발할 수 있습니다.
> **해결책**: `pt-online-schema-change` (Percona), `gh-ost` (GitHub) 같은 온라인 스키마 변경 도구를 사용하거나, 서비스 점검 시간을 활용해야 합니다.

#ALTER #테이블변경 #인덱스 #index #온라인스키마변경

---

### 3. 테이블 삭제 및 초기화 (`DROP`, `TRUNCATE`)

- **`DROP TABLE table_name;`**
  - 테이블의 **구조와 데이터 모두**를 완전히 삭제합니다.
  - 되돌릴 수 없으므로 매우 신중하게 사용해야 합니다.
- **`TRUNCATE TABLE table_name;`**
  - 테이블의 **모든 데이터만** 삭제하고, 구조는 남겨둡니다.
  - `DELETE`보다 훨씬 빠르지만, `ROLLBACK`이 불가능하고 `AUTO_INCREMENT` 값이 초기화됩니다.

#DROP #TRUNCATE #테이블삭제 #테이블초기화
