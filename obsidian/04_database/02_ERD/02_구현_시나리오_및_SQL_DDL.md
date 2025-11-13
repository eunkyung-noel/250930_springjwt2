# 02*구현*시나리오\_및\_SQL_DDL

#유스케이스 #usecase #시나리오 #scenario #DDL #데이터정의어

---

## 🎯 학습 목표

- 설계된 ERD를 바탕으로 실제 애플리케이션의 유스케이스를 정의합니다.
- 각 유스케이스에 따른 데이터베이스 동작(CRUD)을 파악합니다.
- ERD를 실제 데이터베이스에 생성할 수 있는 SQL DDL 스크립트를 작성합니다.

---

### 실제 구현 시나리오 및 유스케이스

#### 시나리오 1: 사용자 회원가입 및 로그인

1.  **유스케이스**: 사용자가 이메일과 비밀번호로 **회원가입**합니다.
2.  **DB 동작 (Create)**: `USER` 테이블에 새로운 사용자의 정보를 `INSERT`합니다. `password`는 반드시 해시하여 `password_hash`에 저장합니다.
3.  **유스케이스**: 등록된 이메일과 비밀번호로 **로그인**합니다.
4.  **DB 동작 (Read)**: `email`로 `USER` 테이블을 `SELECT`하여 사용자를 찾고, 저장된 `password_hash`와 입력된 비밀번호의 해시값을 비교합니다.

#회원가입 #signup #로그인 #login #CRUD

#### 시나리오 2: 공연 정보 조회 및 티켓 구매

1.  **유스케이스**: 사용자가 앱에서 **공연 목록을 조회**합니다.
2.  **DB 동작 (Read)**: `EVENT` 테이블에서 모든 공연 정보를 `SELECT`합니다.
3.  **유스케이스**: 사용자가 공연의 **좌석 현황을 확인**하고 원하는 좌석을 선택합니다.
4.  **DB 동작 (Read)**: `event_id`로 `SEAT` 테이블을 `SELECT`하여 `status`가 'available'인 좌석만 사용자에게 보여줍니다.
5.  **유스케이스**: 사용자가 선택한 좌석을 **예매**하고 결제를 진행합니다.
6.  **DB 동작 (Update, Create)**:
    - **(Transaction 시작)**
    - `SEAT` 테이블에서 선택된 좌석의 `status`를 'reserved'로 `UPDATE`하여 다른 사용자가 선택하지 못하게 합니다. (동시성 제어)
    - 결제가 성공하면, `TICKET` 테이블에 예매 정보를 `INSERT`합니다.
    - `PAYMENT` 테이블에 결제 정보를 `INSERT`합니다.
    - `SEAT` 테이블의 `status`를 'sold'로 최종 `UPDATE`합니다.
    - **(Transaction 종료 - COMMIT)**
    - 만약 중간에 실패하면 **ROLLBACK**하여 모든 변경사항을 되돌립니다.

#동시성제어 #concurrencycontrol #트랜잭션 #transaction #COMMIT #ROLLBACK

#### 시나리오 3: 구매한 티켓 확인 및 취소

1.  **유스케이스**: 사용자가 **마이페이지에서 구매한 티켓 목록**을 확인합니다.
2.  **DB 동작 (Read)**: `user_id`로 `TICKET` 테이블을 `SELECT`하고, `EVENT`, `SEAT` 테이블과 `JOIN`하여 공연명, 좌석번호 등 상세 정보를 함께 조회합니다.
3.  **유스케이스**: 사용자가 **티켓을 취소**합니다.
4.  **DB 동작 (Update, Delete)**:
    - **(Transaction 시작)**
    - `ticket_id`로 `PAYMENT` 테이블의 상태를 'refunded'로 `UPDATE`하고 환불을 처리합니다.
    - `TICKET` 테이블에서 해당 티켓을 `DELETE`합니다.
    - `SEAT` 테이블에서 해당 좌석의 `status`를 다시 'available'로 `UPDATE`합니다.
    - **(Transaction 종료 - COMMIT/ROLLBACK)**

#JOIN #조인 #환불 #refund

---

### SQL DDL (MySQL 8.0 기준)

```sql
-- Event Ticketing System Database Schema for MySQL 8.0

CREATE TABLE `EVENT` (
    `event_id` INT AUTO_INCREMENT NOT NULL,
    `event_name` VARCHAR(255) NOT NULL,
    `event_date` DATETIME NOT NULL,
    `venue` VARCHAR(255) NOT NULL,
    `description` TEXT,
    PRIMARY KEY (`event_id`)
);

CREATE TABLE `USER` (
    `user_id` INT AUTO_INCREMENT NOT NULL,
    `username` VARCHAR(50) UNIQUE NOT NULL,
    `email` VARCHAR(100) UNIQUE NOT NULL,
    `password_hash` VARCHAR(255) NOT NULL,
    `phone_number` VARCHAR(20),
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`user_id`)
);

CREATE TABLE `SEAT` (
    `seat_id` INT AUTO_INCREMENT NOT NULL,
    `seat_number` VARCHAR(10) NOT NULL,
    `section` VARCHAR(50) NOT NULL,
    `status` ENUM('available', 'reserved', 'sold') DEFAULT 'available',
    `event_id` INT NOT NULL,
    PRIMARY KEY (`seat_id`),
    FOREIGN KEY (`event_id`) REFERENCES `EVENT`(`event_id`) ON DELETE CASCADE,
    UNIQUE KEY `unique_seat_per_event` (`event_id`, `seat_number`, `section`)
);

CREATE TABLE `TICKET` (
    `ticket_id` INT AUTO_INCREMENT NOT NULL,
    `purchase_date` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `price` DECIMAL(10,2) NOT NULL,
    `user_id` INT NOT NULL,
    `event_id` INT NOT NULL,
    `seat_id` INT NOT NULL,
    PRIMARY KEY (`ticket_id`),
    FOREIGN KEY (`user_id`) REFERENCES `USER`(`user_id`),
    FOREIGN KEY (`event_id`) REFERENCES `EVENT`(`event_id`),
    FOREIGN KEY (`seat_id`) REFERENCES `SEAT`(`seat_id`),
    UNIQUE KEY `unique_seat_ticket` (`seat_id`)
);

CREATE TABLE `PAYMENT` (
    `payment_id` INT AUTO_INCREMENT NOT NULL,
    `amount` DECIMAL(10,2) NOT NULL,
    `payment_method` ENUM('credit_card', 'bank_transfer') NOT NULL,
    `payment_status` ENUM('pending', 'completed', 'failed', 'refunded') DEFAULT 'pending',
    `payment_date` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `ticket_id` INT NOT NULL,
    PRIMARY KEY (`payment_id`),
    FOREIGN KEY (`ticket_id`) REFERENCES `TICKET`(`ticket_id`)
);
```

#SQL #DDL #MySQL #스키마 #schema
