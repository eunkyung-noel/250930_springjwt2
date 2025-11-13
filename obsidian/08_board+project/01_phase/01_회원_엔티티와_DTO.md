# 1단계: 인증 및 기본 CRUD

## 1. 회원 엔티티와 DTO

프로젝트의 핵심 데이터 모델인 `UserAccount` 엔티티와 계층 간 데이터 전송을 담당하는 DTO(Data Transfer Object)를 설계합니다.

### 1. `BaseEntity` - 공통 필드 관리

대부분의 엔티티는 생성일(`createdAt`), 수정일(`updatedAt`)과 같은 메타데이터를 공통으로 가집니다. `BaseEntity` 클래스를 만들어두면, 다른 엔티티들이 이 클래스를 상속받아 중복 코드를 줄이고 생성/수정 시간을 자동으로 관리할 수 있습니다.

- `@MappedSuperclass`: 이 클래스가 엔티티의 공통 매핑 정보를 포함하는 부모 클래스임을 나타냅니다. 테이블은 생성되지 않습니다.
- `@EntityListeners(AuditingEntityListener.class)`: JPA Auditing 기능을 활성화하여 엔티티의 변경 사항을 감지하고, `@CreatedDate`, `@LastModifiedDate` 어노테이션이 붙은 필드를 자동으로 업데이트합니다.
- `@CreatedDate`, `@LastModifiedDate`: 엔티티가 생성되거나 수정될 때 현재 시간을 자동으로 저장합니다.

```java
// src/main/java/com/example/boardpjt/model/entity/BaseEntity.java

package com.example.boardpjt.model.entity;

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false, nullable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;
}
```

> **💡 JPA Auditing 활성화** > `BaseEntity`의 Auditing 기능이 동작하려면, 메인 애플리케이션 클래스에 `@EnableJpaAuditing` 어노테이션을 추가해야 합니다.
>
> ```java
> // src/main/java/com/example/boardpjt/BoardpjtApplication.java
>
> @EnableJpaAuditing
> @SpringBootApplication
> public class BoardpjtApplication {
>     // ...
> }
> ```

### 2. `UserAccount` 엔티티

사용자 정보를 데이터베이스 테이블과 매핑하는 `UserAccount` 엔티티입니다. `BaseEntity`를 상속받아 생성/수정 시간을 자동으로 관리합니다.

- `@Entity`: 이 클래스가 JPA 엔티티임을 나타냅니다.
- `@Id`, `@GeneratedValue`: `id` 필드가 기본 키(Primary Key)이며, 데이터베이스가 자동으로 값을 생성(Identity 전략)하도록 설정합니다.
- `@Column`: 각 필드를 테이블의 컬럼과 매핑합니다. `nullable = false`는 `NOT NULL` 제약조건을, `unique = true`는 `UNIQUE` 제약조건을 의미합니다.
- **`password` 필드**: 보안을 위해 절대 평문으로 저장해서는 안 됩니다. Spring Security의 `PasswordEncoder`를 통해 해시된 값을 저장해야 합니다.
- **`role` 필드**: 사용자의 권한을 저장합니다. (예: "ROLE_USER", "ROLE_ADMIN")

```java
// src/main/java/com/example/boardpjt/model/entity/UserAccount.java

package com.example.boardpjt.model.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class UserAccount extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 50)
    private String username;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false, length = 50)
    private String role;
}
```

### 3. `UserAccountDTO`

DTO(Data Transfer Object)는 계층 간(Controller, Service, Repository) 데이터 전송을 위해 사용하는 객체입니다. 엔티티를 직접 노출하는 대신 DTO를 사용하면 다음과 같은 장점이 있습니다.

- **보안**: 엔티티의 민감한 정보(예: `password`)를 외부로 노출하지 않을 수 있습니다.
- **유연성**: API 스펙 변경 시 엔티티와 독립적으로 DTO만 수정하면 되므로 유연하게 대처할 수 있습니다.
- **데이터 최적화**: 화면에 필요한 데이터만 담아 전송하므로 네트워크 부하를 줄일 수 있습니다.

#### `Request` DTO

클라이언트(예: 회원가입 폼)에서 서버로 데이터를 요청할 때 사용합니다.

```java
// src/main/java/com/example/boardpjt/model/dto/UserAccountDTO.java

package com.example.boardpjt.model.dto;

import lombok.Getter;
import lombok.Setter;

public class UserAccountDTO {

    @Getter
    @Setter
    public static class Request {
        private String username;
        private String password;
    }
    // ... Response DTO
}
```

#### `Response` DTO

서버가 클라이언트로 데이터를 응답할 때 사용합니다. Java 14부터 도입된 `record` 타입을 사용하면 불변(immutable) 객체를 간결하게 만들 수 있습니다.

```java
// src/main/java/com/example/boardpjt/model/dto/UserAccountDTO.java

public class UserAccountDTO {
    // ... Request DTO

    public record Response(
            Long id,
            String username,
            String role,
            String createdAt
    ) {}
}
```

이제 데이터 모델링이 완료되었습니다. 다음 장에서는 이 엔티티를 데이터베이스에 저장하고 관리하기 위한 `Repository`와 Spring Security 설정을 진행합니다.
