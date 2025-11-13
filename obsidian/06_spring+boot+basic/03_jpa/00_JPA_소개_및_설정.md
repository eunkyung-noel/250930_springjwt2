# JPA 소개 및 설정

## 1. JPA, ORM, 그리고 Spring Data JPA

### 1.1. ORM (Object-Relational Mapping)

ORM은 객체 지향 프로그래밍 언어(예: Java)의 **객체(Object)**와 관계형 데이터베이스(RDB)의 **테이블(Relation)** 사이의 불일치를 해결해주는 기술입니다. 개발자는 SQL 쿼리를 직접 작성하는 대신, 익숙한 객체지향 코드를 통해 데이터베이스를 조작할 수 있습니다.

- **장점**:
  - **생산성 향상**: 반복적인 SQL 작성 없이 비즈니스 로직에 집중할 수 있습니다.
  - **유지보수 용이**: SQL 종속성이 줄어들어 코드의 가독성과 유지보수성이 향상됩니다.
  - **데이터베이스 독립성**: 특정 데이터베이스에 종속되지 않는 코드를 작성할 수 있습니다. (예: MySQL -> PostgreSQL 전환 용이)

### 1.2. JPA (Java Persistence API)

JPA는 자바 진영의 **ORM 표준 명세(Specification)**입니다. 즉, 인터페이스의 모음이며 실제 구현이 아닙니다. 개발자는 JPA 표준에 맞춰 코드를 작성하고, 그 뒤에서 동작하는 실제 구현체는 선택할 수 있습니다.

- **Hibernate**: 가장 널리 사용되는 JPA 구현체입니다. JPA 표준 기능 외에도 캐싱, 성능 최적화 등 다양한 부가 기능을 제공합니다.

### 1.3. Spring Data JPA

Spring Data JPA는 JPA를 더 쉽고 편리하게 사용할 수 있도록 스프링 프레임워크가 제공하는 모듈입니다. JPA가 ORM의 '명세'라면, Spring Data JPA는 이를 '추상화'하여 한 단계 더 발전시킨 것입니다.

- **핵심 기능**: `JpaRepository` 인터페이스를 제공하여, 개발자가 인터페이스만 선언해도 런타임에 기본적인 CRUD(Create, Read, Update, Delete) 메서드를 자동으로 생성해줍니다.
- **장점**:
  - **보일러플레이트 코드 제거**: DAO(Data Access Object)나 Repository 구현 클래스를 직접 작성할 필요가 없습니다.
  - **쿼리 메서드**: 정해진 규칙에 따라 메서드 이름을 작성하면(예: `findByName(String name)`), 해당 쿼리를 자동으로 생성합니다.
  - **페이징 및 정렬**: `Pageable` 인터페이스를 통해 페이징과 정렬을 간편하게 처리할 수 있습니다.

> **면접 포인트**
>
> - **JPA와 Hibernate의 관계**: JPA는 '명세(인터페이스)'이고, Hibernate는 그 명세를 구현한 '구현체(클래스)'입니다.
> - **Spring Data JPA의 역할**: JPA를 기반으로 Repository 계층을 추상화하여, CRUD 코드를 자동 생성하고 개발 생산성을 극대화하는 역할을 합니다.

---

## 2. 프로젝트 설정

Spring Boot 프로젝트에서 Spring Data JPA를 사용하기 위한 기본 설정입니다.

### 2.1. `build.gradle` 의존성 추가

`spring-boot-starter-data-jpa`와 데이터베이스 드라이버(예: H2, MySQL)를 추가합니다.

```groovy
// build.gradle
plugins {
    id 'org.springframework.boot' version '3.2.5' // 버전에 맞게 사용
    id 'io.spring.dependency-management' version '1.1.4'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Data JPA
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    // Spring Web (Controller 등 사용 시)
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // Thymeleaf (View 템플릿 엔진 사용 시)
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'

    // Lombok (보일러플레이트 코드 제거)
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // H2 Database (개발 및 테스트용 인메모리 DB)
    runtimeOnly 'com.h2database:h2'
    // MySQL Driver (운영 환경용)
    // runtimeOnly 'com.mysql:mysql-connector-j'

    // Test
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### 2.2. `application.properties` 또는 `application.yml` 설정

데이터베이스 연결 정보와 JPA 관련 설정을 추가합니다. 개발 초기에는 인메모리 DB인 H2를 사용하여 빠르게 개발하고, 이후 외부 DB로 전환하는 것이 일반적입니다.

#### `application.properties` 사용 시

```properties
# src/main/resources/application.properties

# H2 데이터베이스 설정
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 콘솔 활성화 (개발 편의 기능)
# 브라우저에서 http://localhost:8080/h2-console 로 접속하여 DB 확인 가능
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA/Hibernate 설정
# ddl-auto: 애플리케이션 실행 시점에 엔티티를 기반으로 DB 스키마를 자동 처리하는 전략
# - create: 기존 테이블 삭제 후 다시 생성 (테스트 환경)
# - update: 변경된 스키마만 반영 (개발 환경)
# - validate: 엔티티와 테이블이 정상 매핑되었는지 검증 (운영 환경)
# - none: 아무것도 하지 않음 (운영 환경)
spring.jpa.hibernate.ddl-auto=create

# 실행되는 SQL 쿼리를 로그로 출력
spring.jpa.show-sql=true

# SQL 로그를 보기 좋게 포맷팅
spring.jpa.properties.hibernate.format_sql=true
```

#### `application.yml` 사용 시 (권장)

YAML은 계층 구조를 통해 가독성이 더 좋습니다.

```yaml
# src/main/resources/application.yml

spring:
  # 데이터베이스 설정
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  # H2 콘솔 설정
  h2:
    console:
      enabled: true
      path: /h2-console

  # JPA 설정
  jpa:
    # DDL(Data Definition Language) 자동 생성 전략
    hibernate:
      ddl-auto: create
    # SQL 쿼리 로깅
    show-sql: true
    properties:
      hibernate:
        # SQL 포맷팅
        format_sql: true
        # N+1 문제 완화를 위한 배치 사이즈 (추후 설명)
        default_batch_fetch_size: 100
```

이제 프로젝트에서 Spring Data JPA를 사용할 준비가 완료되었습니다. 다음 단계에서는 데이터베이스 테이블과 매핑될 **엔티티(Entity)**를 정의하는 방법을 알아봅니다.
