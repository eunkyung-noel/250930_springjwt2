# 04. 실습: 첫 Spring Boot 애플리케이션

#springboot #실습 #restcontroller #getmapping

`start.spring.io`를 통해 생성한 프로젝트를 기반으로, "Hello, Spring Boot!"를 출력하는 간단한 웹 애플리케이션을 만들어 보겠습니다. 이 실습을 통해 Spring Boot의 기본 구조와 동작 방식을 이해할 수 있습니다.

---

## 1. 프로젝트 기본 구조

`start.spring.io`에서 생성된 프로젝트는 다음과 같은 기본 구조를 가집니다.

```
src/main/java
 └─ com/example/firstboot
    └─ FirstbootApplication.java  // 메인 애플리케이션 클래스

src/main/resources
 ├─ static/                     // 정적 리소스 (CSS, JS, 이미지 등)
 ├─ templates/                  // 템플릿 파일 (Thymeleaf 등)
 └─ application.properties      // 애플리케이션 설정 파일

pom.xml 또는 build.gradle          // 빌드 설정 파일
```

---

## 2. 메인 애플리케이션 클래스

`FirstbootApplication.java`는 애플리케이션의 진입점(Entry Point) 역할을 합니다.

```java
// FirstbootApplication.java
package com.example.firstboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @SpringBootApplication: 이 애너테이션 하나로 세 가지 주요 기능을 수행합니다.
 * 1. @SpringBootConfiguration: 이 클래스를 스프링 부트의 설정 클래스로 지정합니다.
 * 2. @EnableAutoConfiguration: 클래스패스를 기반으로 설정을 자동으로 구성합니다.
 * 3. @ComponentScan: 현재 패키지(com.example.firstboot) 및 하위 패키지를 스캔하여
 *    @Component, @Service, @RestController 등의 빈(Bean)을 등록합니다.
 */
@SpringBootApplication
public class FirstbootApplication {

    public static void main(String[] args) {
        // SpringApplication.run()을 통해 내장 웹 서버(Tomcat)를 실행하고
        // 스프링 부트 애플리케이션을 구동합니다.
        SpringApplication.run(FirstbootApplication.class, args);
    }
}
```

---

## 3. 컨트롤러(Controller) 작성

클라이언트의 웹 요청을 받아 처리하는 컨트롤러를 작성해 보겠습니다.

### 가. `HelloController.java`

`/hello` 경로로 GET 요청이 오면 "Hello, Spring Boot!" 문자열을 반환하는 간단한 컨트롤러입니다.

```java
// HelloController.java
package com.example.firstboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @RestController: 이 클래스가 RESTful 웹 서비스의 컨트롤러임을 나타냅니다.
 * 이 애너테이션이 붙은 클래스의 모든 메서드는 @ResponseBody를 포함하게 되어,
 * 뷰(View)를 반환하는 대신 데이터(주로 JSON 또는 문자열) 자체를 HTTP 응답 본문에 직접 씁니다.
 */
@RestController
public class HelloController {

    /**
     * @GetMapping("/hello"): HTTP GET 요청을 특정 경로("/hello")에 매핑합니다.
     * 사용자가 브라우저에서 'http://localhost:8080/hello'로 접속하면 이 메서드가 호출됩니다.
     */
    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
```

### 나. `@RequestMapping`을 사용한 컨트롤러

`@RequestMapping`을 클래스 레벨에 적용하여 공통 URL 경로를 지정할 수도 있습니다.

```java
// ByeController.java
package com.example.firstboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @RequestMapping("/bye"): 이 클래스 내의 모든 매핑은 "/bye"라는 상위 경로를 갖게 됩니다.
 */
@RestController
@RequestMapping("/bye")
public class ByeController {

    /**
     * @GetMapping: 클래스 레벨의 @RequestMapping 경로에 이어집니다.
     * 따라서 이 메서드는 'http://localhost:8080/bye' 경로의 GET 요청을 처리합니다.
     */
    @GetMapping
    public String bye() {
        return "Bye, Spring Boot!";
    }
}
```

---

## 4. 애플리케이션 실행 및 확인

1.  **실행**: IDE에서 `FirstbootApplication.java` 파일을 열고 `main` 메서드를 실행합니다.
2.  **로그 확인**: 콘솔에 Spring Boot 로고와 함께 내장 Tomcat 서버가 8080 포트에서 시작되었다는 로그가 나타납니다.
3.  **브라우저 확인**:
    - 웹 브라우저를 열고 `http://localhost:8080/hello`로 접속하면 "Hello, Spring Boot!" 메시지가 표시됩니다.
    - `http://localhost:8080/bye`로 접속하면 "Bye, Spring Boot!" 메시지가 표시됩니다.

---

## 5. 포트 변경하기

만약 8080 포트가 이미 사용 중이거나 다른 포트를 사용하고 싶다면, `src/main/resources/application.properties` 파일에 다음 한 줄을 추가하여 포트를 변경할 수 있습니다.

```properties
# application.properties

# 서버 포트를 8888로 변경
server.port=8888
```

이제 애플리케이션을 다시 시작하면 `http://localhost:8888`에서 실행되는 것을 확인할 수 있습니다. 이처럼 Spring Boot는 간단한 설정만으로 애플리케이션의 동작을 쉽게 변경할 수 있는 강력한 기능을 제공합니다.
