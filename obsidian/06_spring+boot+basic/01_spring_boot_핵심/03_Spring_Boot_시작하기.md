# 03. Spring Boot 시작하기

#springboot #스프링부트 #starter #스타터 #autoconfiguration #자동설정

Spring Boot는 Spring Framework를 기반으로, 최소한의 설정으로 독립 실행 가능한(stand-alone) 프로덕션 등급의 Spring 기반 애플리케이션을 쉽게 만들 수 있도록 도와주는 도구입니다. 복잡한 초기 설정을 대신 처리해주어 개발자가 비즈니스 로직에만 집중할 수 있게 해줍니다.

---

## 1. Spring Boot의 핵심 목표

- **빠른 개발 경험**: 복잡한 XML 설정이나 번거로운 초기 구성 없이 바로 개발을 시작할 수 있습니다.
- **"의견을 가진(Opinionated)" 기본 설정**: 널리 사용되는 설정들을 기본값으로 제공하여, 개발자가 특별히 신경 쓰지 않아도 대부분의 상황에서 잘 동작하도록 합니다.
- **내장 서버**: Tomcat, Jetty, Undertow 같은 웹 서버를 내장하고 있어, 별도의 서버 설치 없이 JAR 파일만으로 애플리케이션을 실행할 수 있습니다.
- **프로덕션 준비 기능**: 헬스 체크(health checks), 메트릭(metrics) 수집 등 운영에 필요한 기능들을 기본적으로 제공합니다.

---

## 2. 스타터(Starters)

- **정의**: 특정 기능을 개발하는 데 필요한 의존성들의 묶음(bundle)입니다.
- **역할**: 개발자는 `spring-boot-starter-web`과 같은 스타터 하나만 의존성에 추가하면, Spring MVC, 내장 Tomcat, Jackson(JSON 라이브러리) 등 웹 개발에 필요한 수많은 라이브러리들이 **전이 의존성(transitive dependencies)**으로 자동 포함됩니다. 이를 통해 개발자는 호환되는 라이브러리 버전을 일일이 찾고 관리할 필요가 없어집니다.

### 대표적인 스타터 예시

- **`spring-boot-starter-web`**: Spring MVC를 사용하여 웹 애플리케이션 및 RESTful API를 구축하기 위한 스타터.
- **`spring-boot-starter-data-jpa`**: Spring Data JPA와 Hibernate를 사용하여 데이터베이스와 상호작용하기 위한 스타터.
- **`spring-boot-starter-test`**: JUnit, Mockito 등 테스트에 필요한 라이브러리들을 포함하는 스타터.

**Gradle 의존성 예시 (`build.gradle`):**

```groovy
dependencies {
    // 이 한 줄로 Spring MVC, 내장 톰캣 등 웹 개발에 필요한 모든 의존성이 자동으로 추가됩니다.
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

---

## 3. 자동 설정 (Auto-configuration)

- **정의**: Spring Boot의 가장 강력한 기능 중 하나로, 클래스패스(classpath)에 있는 라이브러리들을 감지하여 관련 설정을 자동으로 구성해주는 메커니즘입니다.
- **예시**: 클래스패스에 `spring-boot-starter-web`이 있으면, Spring Boot는 이것이 웹 애플리케이션이라고 판단하고 `DispatcherServlet`, `Tomcat` 서버 등 웹 개발에 필요한 빈(Bean)들을 자동으로 설정하고 등록합니다.
- **동작 원리**: `@SpringBootApplication` 애너테이션 안에 있는 `@EnableAutoConfiguration`이 이 기능을 활성화하며, 조건부 설정(`@ConditionalOn...`)을 통해 특정 조건이 만족될 때만 설정이 적용되도록 합니다. 개발자는 필요에 따라 `application.properties`나 `application.yml` 파일에서 기본 설정을 덮어쓸 수(override) 있습니다.

---

## 4. 프로젝트 생성: `start.spring.io`

[start.spring.io](https://start.spring.io/)는 Spring Boot 프로젝트의 기본 구조를 생성해주는 공식 웹 도구입니다.

### 사용 순서

1.  **Project**: 빌드 도구 선택 (Maven 또는 Gradle)
2.  **Language**: 프로그래밍 언어 선택 (Java, Kotlin, Groovy)
3.  **Spring Boot 버전**: 최신 안정 버전 또는 LTS(Long-Term Support) 버전 선택 (예: 3.3.x)
4.  **Project Metadata**: Group, Artifact 등 프로젝트 정보 입력
5.  **Packaging**: 실행 파일 형식 선택 (Jar 또는 War, 보통 Jar 선택)
6.  **Java 버전**: 사용할 Java 버전 선택
7.  **Dependencies**: 프로젝트에 필요한 스타터 추가 (예: 'Spring Web', 'Lombok', 'Spring Data JPA')
8.  **Generate**: 버튼을 클릭하면 설정이 완료된 프로젝트 압축 파일(ZIP)이 다운로드됩니다.

---

## 5. Spring vs Spring Boot 비교

| 항목            | 순수 Spring                                                                   | Spring Boot                                                                                       |
| --------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **의존성 선언** | 각 모듈의 의존성을 개별적으로, 버전을 맞춰서 직접 선언해야 함                 | **스타터(Starter)** 번들을 통해 관련 의존성들을 한 번에 가져옴                                    |
| **설정량**      | `DataSource`, `DispatcherServlet` 등 모든 것을 Java 설정 또는 XML로 직접 구성 | **자동 설정(Auto-configuration)**으로 대부분의 설정을 자동으로 처리하고, 필요한 부분만 오버라이드 |
| **실행 방식**   | WAR 파일로 빌드하여 외부 WAS(Web Application Server)에 배포                   | **내장 서버(Tomcat 등)**를 포함한 실행 가능한 JAR 파일로 빌드하여 독립적으로 실행                 |
| **개발 경험**   | 초기 구성이 복잡하고 시간이 많이 소요됨                                       | **관례 우선 원칙(Convention over Configuration)**에 따라 즉시 실행 가능하며, 생산성이 높음        |

결론적으로, Spring Boot는 Spring Framework의 강력한 기능들을 그대로 활용하면서, 개발자가 마주하는 복잡한 설정과 환경 구성의 부담을 획기적으로 줄여주는 생산성 도구입니다.
