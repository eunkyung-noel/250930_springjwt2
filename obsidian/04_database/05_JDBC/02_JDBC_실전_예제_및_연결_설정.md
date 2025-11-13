---
tags:
  - java
  - jdbc
  - database
  - mysql
  - maven
  - dotenv
---

# 02. JDBC 실전 예제 및 연결 설정

이전 문서에서는 JDBC의 프로그래밍 흐름과 DAO/DTO 패턴을 살펴보았습니다. 이번에는 실제 코드를 통해 데이터베이스 연결을 설정하고, 간단한 CRUD(Create, Read, Update, Delete) 작업을 수행하는 방법을 구체적으로 알아봅니다.

#학습목표

- Maven을 사용하여 JDBC 드라이버 및 기타 라이브러리 의존성을 관리할 수 있습니다.
- `.env` 파일을 통해 데이터베이스 연결 정보를 안전하게 관리할 수 있습니다.
- JDBC를 사용하여 DDL, DML, DQL을 실행하는 Java 코드를 작성할 수 있습니다.

---

## 1. 프로젝트 설정: Maven 의존성 추가

JDBC를 사용하려면 먼저 해당 데이터베이스의 드라이버가 필요합니다. 또한, 데이터베이스 접속 정보를 코드와 분리하여 안전하게 관리하기 위해 `dotenv-java` 라이브러리를 사용합니다. 이러한 의존성은 Maven의 `pom.xml` 파일을 통해 쉽게 관리할 수 있습니다.

#메이븐 #maven #의존성 #dependency

```xml
<!-- pom.xml -->
<dependencies>
    <!-- MySQL 8.0 Connector -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- .env 파일 관리를 위한 라이브러리 -->
    <dependency>
        <groupId>io.github.cdimascio</groupId>
        <artifactId>dotenv-java</artifactId>
        <version>3.0.0</version>
    </dependency>
</dependencies>
```

- **mysql-connector-j**: MySQL 데이터베이스에 연결하기 위한 공식 JDBC 드라이버입니다.
- **dotenv-java**: `.env` 파일에 저장된 환경 변수를 Java 애플리케이션에서 쉽게 불러올 수 있도록 도와줍니다.

---

## 2. 데이터베이스 연결 정보 관리: `.env`

데이터베이스 접속 정보(URL, 사용자 이름, 비밀번호)를 소스 코드에 직접 하드코딩하는 것은 보안상 매우 위험합니다. 대신, 프로젝트 루트에 `.env` 파일을 생성하여 정보를 관리합니다.

#환경변수 #environment_variable #닷엔브

```env
# .env
DB_URL=jdbc:mysql://localhost:3306/mydatabase
DB_USER=myuser
DB_PASSWORD=mypassword
```

**주의**: `.gitignore` 파일에 `.env`를 추가하여 이 파일이 Git 저장소에 커밋되지 않도록 반드시 설정해야 합니다.

```gitignore
# .gitignore
.env
```

---

## 3. 데이터베이스 연결 유틸리티: `DBUtil`

애플리케이션 전역에서 일관된 방법으로 데이터베이스 연결을 얻을 수 있도록 유틸리티 클래스를 만드는 것이 좋습니다. `DBUtil` 클래스는 `dotenv-java`를 사용하여 `.env` 파일의 정보를 읽어 `Connection` 객체를 생성하고 반환합니다.

#유틸리티클래스 #utility_class #커넥션 #connection

```java
import io.github.cdimascio.dotenv.Dotenv;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBUtil {
    private static final Dotenv dotenv = Dotenv.load();
    private static final String URL = dotenv.get("DB_URL");
    private static final String USER = dotenv.get("DB_USER");
    private static final String PASSWORD = dotenv.get("DB_PASSWORD");

    // 데이터베이스 커넥션을 반환하는 정적 메서드
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}
```

- `Dotenv.load()`: 프로젝트 루트에서 `.env` 파일을 찾아 내용을 로드합니다.
- `dotenv.get("KEY")`: 키에 해당하는 값을 문자열로 가져옵니다.
- `DriverManager.getConnection()`: 로드된 정보를 바탕으로 데이터베이스 연결을 수립합니다.

---

## 4. JDBC 실전 예제

이제 `DBUtil`을 사용하여 실제 데이터베이스 작업을 수행하는 예제 코드를 작성해 보겠습니다.

### 가. 테이블 생성 (DDL)

`Statement`를 사용하여 간단한 `users` 테이블을 생성하는 예제입니다.

#DDL #테이블생성 #create_table

```java
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

public class CreateTableExample {
    public static void main(String[] args) {
        // 1. SQL 쿼리 정의
        String sql = "CREATE TABLE IF NOT EXISTS users (" +
                     "id INT AUTO_INCREMENT PRIMARY KEY, " +
                     "name VARCHAR(50) NOT NULL, " +
                     "email VARCHAR(100) NOT NULL UNIQUE" +
                     ")";

        // 2. try-with-resources 구문으로 자원 자동 해제
        try (Connection conn = DBUtil.getConnection();
             Statement stmt = conn.createStatement()) {

            // 3. DDL 실행
            stmt.executeUpdate(sql);
            System.out.println("테이블 'users'가 성공적으로 생성되었거나 이미 존재합니다.");

        } catch (SQLException e) {
            System.err.println("테이블 생성 중 오류가 발생했습니다: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

- `executeUpdate(sql)`: DDL(CREATE, ALTER, DROP) 또는 DML(INSERT, UPDATE, DELETE) 쿼리를 실행할 때 사용됩니다. 실행 후 영향받은 행의 수를 반환하지만, DDL의 경우 보통 0을 반환합니다.

### 나. 데이터 삽입 (DML)

`PreparedStatement`를 사용하여 `users` 테이블에 새로운 사용자를 추가합니다. SQL 인젝션 공격을 방지하기 위해 항상 `PreparedStatement`를 사용하는 것이 좋습니다.

#DML #데이터삽입 #insert #prepared_statement

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class InsertUserExample {
    public static void main(String[] args) {
        // 1. SQL 쿼리 템플릿 정의
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";

        // 2. try-with-resources
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {

            // 3. 파라미터 바인딩
            pstmt.setString(1, "Alice");
            pstmt.setString(2, "alice@example.com");

            // 4. DML 실행
            int affectedRows = pstmt.executeUpdate();
            System.out.println(affectedRows + "개의 행이 성공적으로 삽입되었습니다.");

        } catch (SQLException e) {
            System.err.println("데이터 삽입 중 오류가 발생했습니다: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 다. 데이터 조회 (DQL)

`PreparedStatement`와 `ResultSet`을 사용하여 삽입된 데이터를 조회하고 콘솔에 출력합니다.

#DQL #데이터조회 #select #result_set

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class SelectUserExample {
    public static void main(String[] args) {
        String sql = "SELECT id, name, email FROM users WHERE name = ?";

        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {

            pstmt.setString(1, "Alice");

            // DQL 실행 및 결과 집합 받기
            try (ResultSet rs = pstmt.executeQuery()) {
                // 결과 집합 순회
                while (rs.next()) {
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    String email = rs.getString("email");
                    System.out.printf("ID: %d, Name: %s, Email: %s\n", id, name, email);
                }
            }
        } catch (SQLException e) {
            System.err.println("데이터 조회 중 오류가 발생했습니다: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

- `executeQuery()`: DQL(SELECT) 쿼리를 실행하고, 그 결과를 `ResultSet` 객체로 반환합니다.
- `rs.next()`: 커서를 다음 행으로 이동시킵니다. 읽을 행이 있으면 `true`, 없으면 `false`를 반환합니다.
- `rs.getXXX("column_name")`: 현재 커서가 가리키는 행의 특정 컬럼 값을 가져옵니다.

다음 문서에서는 여러 SQL 작업을 하나의 논리적 단위로 묶는 **트랜잭션 관리**에 대해 알아보겠습니다.
