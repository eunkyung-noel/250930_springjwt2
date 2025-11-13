---
tags:
  - java
  - jdbc
  - database
  - transaction
  - acid
---

# 03. JDBC 심화: 트랜잭션 관리 및 응용

이전 문서에서는 JDBC를 사용한 기본적인 CRUD 작업 방법을 다루었습니다. 이번에는 여러 데이터베이스 연산을 하나의 논리적 단위로 묶어 처리하는 **트랜잭션(Transaction)** 관리와 그 중요성에 대해 알아봅니다.

#학습목표

- 트랜잭션의 개념과 ACID 원칙을 이해합니다.
- JDBC를 사용하여 트랜잭션을 수동으로 제어(Commit, Rollback)할 수 있습니다.
- 계좌 이체와 같은 실제 시나리오에 트랜잭션을 적용할 수 있습니다.

---

## 1. 트랜잭션이란?

**트랜잭션(Transaction)**은 데이터베이스의 상태를 변화시키기 위해 수행하는 작업의 논리적 단위입니다. 여러 개의 SQL 쿼리(예: `UPDATE`, `INSERT`)가 하나의 트랜잭션으로 묶일 수 있으며, 이 작업들은 **모두 성공하거나 모두 실패**해야 합니다. 이를 'All or Nothing' 원칙이라고 합니다.

#트랜잭션 #transaction #원자성 #atomicity

가장 대표적인 예는 **계좌 이체**입니다.

1. A 계좌에서 5만 원을 차감 (`UPDATE`)
2. B 계좌에 5만 원을 증액 (`UPDATE`)

만약 1번 작업만 성공하고 시스템에 장애가 발생하여 2번 작업이 실패한다면, 5만 원은 공중으로 사라지게 됩니다. 트랜잭션은 이러한 상황을 방지하여 데이터의 **일관성(Consistency)**과 **무결성(Integrity)**을 보장합니다.

### ACID 원칙

트랜잭션은 다음 네 가지 핵심 원칙(ACID)을 보장해야 합니다.

#ACID #에이시드

- **원자성 (Atomicity)**: 트랜잭션에 포함된 모든 작업이 성공적으로 완료되거나, 아니면 아무 작업도 수행되지 않은 상태로 되돌려져야 합니다.
- **일관성 (Consistency)**: 트랜잭션이 성공적으로 완료되면 데이터베이스는 항상 일관된 상태를 유지해야 합니다. (예: 계좌 총액은 이체 전후로 동일)
- **고립성 (Isolation)**: 하나의 트랜잭션이 실행되는 동안 다른 트랜잭션의 영향을 받아서는 안 됩니다. 여러 트랜잭션이 동시에 실행되더라도 마치 순차적으로 실행된 것처럼 동작해야 합니다.
- **지속성 (Durability)**: 성공적으로 완료된 트랜잭션의 결과는 시스템에 영구적으로 저장되어야 하며, 장애가 발생하더라도 데이터가 손실되지 않아야 합니다.

---

## 2. JDBC 트랜잭션 관리

JDBC는 기본적으로 **자동 커밋(Auto-commit)** 모드로 동작합니다. 즉, 각각의 SQL 문이 실행될 때마다 자동으로 트랜잭션이 완료(커밋)됩니다.

계좌 이체와 같이 여러 쿼리를 하나의 논리적 단위로 묶으려면 수동 커밋 모드로 전환해야 합니다.

#수동커밋 #manual_commit #오토커밋 #auto_commit

### 주요 메서드

- `connection.setAutoCommit(false)`: 자동 커밋 모드를 비활성화합니다. 이 시점부터 트랜잭션이 시작됩니다.
- `connection.commit()`: 트랜잭션에 포함된 모든 변경사항을 데이터베이스에 영구적으로 반영합니다.
- `connection.rollback()`: 트랜잭션 도중 오류가 발생했을 때, 트랜잭션 시작 이전 상태로 모든 변경사항을 되돌립니다.

### 트랜잭션 처리 흐름도

```mermaid
graph TD
    A[트랜잭션 시작<br>setAutoCommit(false)] --> B{작업 1 수행};
    B --> C{작업 2 수행};
    C --> D{...};
    D --> E{모든 작업 성공?};
    E -- Yes --> F[커밋<br>commit()];
    E -- No --> G[롤백<br>rollback()];
    F --> H[트랜잭션 종료];
    G --> H;
```

---

## 3. 계좌 이체 예제 코드

두 계좌 간의 금액을 이체하는 시나리오를 통해 JDBC 트랜잭션 관리를 실제로 구현해 보겠습니다.

- `accounts` 테이블이 있다고 가정합니다. (`id`, `account_holder`, `balance`)

#계좌이체 #bank_transfer #예제 #example

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class BankTransferExample {

    public static void main(String[] args) {
        Connection conn = null;
        try {
            // 1. 데이터베이스 연결 및 수동 커밋 모드 설정
            conn = DBUtil.getConnection();
            conn.setAutoCommit(false); // 트랜잭션 시작

            // 2. 출금 계좌에서 금액 차감
            String withdrawSQL = "UPDATE accounts SET balance = balance - ? WHERE id = ?";
            try (PreparedStatement pstmt1 = conn.prepareStatement(withdrawSQL)) {
                pstmt1.setDouble(1, 100.00); // 이체할 금액
                pstmt1.setInt(2, 1);         // 출금 계좌 ID
                int rowsAffected1 = pstmt1.executeUpdate();
                if (rowsAffected1 == 0) {
                    throw new SQLException("출금 계좌를 찾을 수 없거나 잔액이 부족합니다.");
                }
            }

            // 3. 입금 계좌에 금액 증액
            String depositSQL = "UPDATE accounts SET balance = balance + ? WHERE id = ?";
            try (PreparedStatement pstmt2 = conn.prepareStatement(depositSQL)) {
                pstmt2.setDouble(1, 100.00); // 이체할 금액
                pstmt2.setInt(2, 2);         // 입금 계좌 ID
                int rowsAffected2 = pstmt2.executeUpdate();
                if (rowsAffected2 == 0) {
                    throw new SQLException("입금 계좌를 찾을 수 없습니다.");
                }
            }

            // 4. 모든 작업이 성공하면 커밋
            conn.commit();
            System.out.println("계좌 이체가 성공적으로 완료되었습니다.");

        } catch (SQLException e) {
            // 5. 오류 발생 시 롤백
            System.err.println("오류가 발생하여 트랜잭션을 롤백합니다: " + e.getMessage());
            if (conn != null) {
                try {
                    conn.rollback();
                    System.out.println("롤백이 완료되었습니다.");
                } catch (SQLException ex) {
                    System.err.println("롤백 중 오류가 발생했습니다: " + ex.getMessage());
                }
            }
        } finally {
            // 6. 자원 해제
            if (conn != null) {
                try {
                    conn.setAutoCommit(true); // 커넥션 풀에 반환하기 전 기본 상태로 복원
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 코드 설명

1.  **`conn.setAutoCommit(false)`**: 트랜잭션을 시작합니다.
2.  **출금 및 입금 `UPDATE` 실행**: 각 `executeUpdate()`가 성공했는지 확인합니다. 만약 영향받은 행이 0이라면, 해당 계좌가 없거나 조건이 맞지 않는 경우이므로 예외를 발생시켜 트랜잭션을 중단합니다.
3.  **`conn.commit()`**: 두 `UPDATE` 문이 모두 성공적으로 실행되었을 때만 호출되어 변경사항을 최종 확정합니다.
4.  **`conn.rollback()`**: `try` 블록 내에서 `SQLException`이 발생하면 `catch` 블록으로 이동하여 `rollback()`을 호출합니다. 이로써 출금 작업만 반영되는 등의 부분적인 데이터 변경을 막을 수 있습니다.
5.  **`finally` 블록**: 예외 발생 여부와 관계없이 항상 실행됩니다. 커넥션을 닫기 전에 `setAutoCommit(true)`를 호출하여 커넥션 풀에 반환될 연결 객체를 기본 상태로 되돌리는 것이 좋은 습관입니다.

이것으로 JDBC의 기본부터 심화 과정까지의 여정을 마칩니다. 다음 장에서는 웹의 기초가 되는 HTML에 대해 학습합니다.
