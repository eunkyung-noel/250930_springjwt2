# JSP 연동과 데이터 전달

#JSP #JSP연동 #데이터전달 #Model #컨트롤러 #Controller #뷰 #View

Spring Boot에서 JSP를 뷰 템플릿으로 사용하기 위해서는 몇 가지 설정이 필요합니다. Spring Boot 3.x부터는 JAR 패키징 방식에서 JSP 사용을 권장하지 않으므로, **WAR 패키징 방식**으로 프로젝트를 구성해야 합니다.

---

## 1. `build.gradle` 설정 (WAR 패키징 기준)

JSP를 사용하기 위해 `build.gradle` 파일에 다음과 같이 의존성을 추가하고 WAR 플러그인을 적용해야 합니다.

1.  **패키징 방식 변경**: `plugins` 블록에 `war` 플러그인을 추가합니다.
2.  **JSP 엔진 추가**: JSP 파일을 서블릿으로 변환하고 컴파일하는 `tomcat-embed-jasper` 의존성을 추가합니다.
3.  **외부 WAS 배포 설정**: 내장 톰캣을 사용하지만, 외부 WAS 환경에도 배포할 수 있도록 `providedRuntime` 설정을 추가합니다.

```groovy
// build.gradle

plugins {
    id 'java'
    id 'war' // 1. JAR 대신 WAR 파일을 빌드하도록 설정
    id 'org.springframework.boot' version '3.2.8' // 버전에 맞게 사용
    id 'io.spring.dependency-management' version '1.1.5'
}

// ... (group, version, java, repositories 등)

dependencies {
    // 웹 개발에 필요한 spring-webmvc, 내장 톰캣 등을 포함
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // 2. JSP 엔진(Jasper) 의존성 추가
    implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'

    // 3. WAR 파일을 외부 Tomcat에 배포할 때 필요
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'

    // JSTL(JSP Standard Tag Library) 사용을 위한 의존성 (다음 장에서 사용)
    implementation 'jakarta.servlet.jsp.jstl:jakarta.servlet.jsp.jstl-api'
    implementation 'org.glassfish.web:jakarta.servlet.jsp.jstl'
}
```

---

## 2. `application.properties` 설정

`ViewResolver`가 JSP 파일의 위치를 올바르게 찾을 수 있도록 `application.properties` 파일에 경로와 확장자를 설정해야 합니다.

- **JSP 파일 위치**: JSP 파일은 보안상의 이유로 클라이언트가 직접 URL로 접근할 수 없는 `src/main/webapp/WEB-INF/` 경로 아래에 두는 것이 관례입니다.
- **설정 내용**:
  - `spring.mvc.view.prefix`: 컨트롤러가 반환하는 뷰 이름 앞에 붙일 경로
  - `spring.mvc.view.suffix`: 컨트롤러가 반환하는 뷰 이름 뒤에 붙일 확장자

```properties
# src/main/resources/application.properties

# ViewResolver 설정
# 컨트롤러가 "hello"를 반환하면 "/WEB-INF/views/hello.jsp" 파일을 찾게 됨
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

---

## 3. Controller에서 View로 데이터 전달하기

Controller는 `Model` 객체를 사용하여 View(JSP)로 데이터를 전달합니다. `Model` 객체는 컨트롤러 메서드의 파라미터로 선언하기만 하면 스프링이 자동으로 주입해 줍니다.

- **데이터 추가**: `model.addAttribute("키", "값")` 형태로 데이터를 추가합니다.
- **뷰 반환**: 뷰의 논리적 이름(예: `"greet"`)을 문자열로 반환합니다.

### 예제 코드

#### `HelloController.java`

```java
package com.example.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller // 이 클래스는 웹 요청을 처리하는 컨트롤러임을 명시
public class HelloController {

    @GetMapping("/greeting") // HTTP GET /greeting 요청을 이 메서드와 매핑
    public String greet(Model model) {
        // Model 객체에 "username"과 "message" 키로 데이터를 담음
        model.addAttribute("username", "홍길동");
        model.addAttribute("message", "오늘도 좋은 하루 되세요!");

        // ViewResolver에게 "greet"라는 뷰 이름을 전달
        // 최종적으로 /WEB-INF/views/greet.jsp 파일을 찾아 렌더링함
        return "greet";
    }
}
```

#### `greet.jsp`

JSP에서는 스크립틀릿의 표현식(`<%= ... %>`)이나 EL(Expression Language, `${...}`)을 사용하여 컨트롤러에서 전달받은 데이터를 출력할 수 있습니다. EL을 사용하는 것이 더 간결하고 권장됩니다.

```jsp
<%-- /src/main/webapp/WEB-INF/views/greet.jsp --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Greeting</title>
</head>
<body>
    <%-- 방법 1: 스크립틀릿 표현식 사용 --%>
    <h1>안녕하세요, <%= request.getAttribute("username") %>님!</h1>

    <%-- 방법 2: EL(Expression Language) 사용 (권장) --%>
    <p>${message}</p>
</body>
</html>
```

이렇게 설정하면, 클라이언트가 `/greeting`으로 요청했을 때 `HelloController`가 `Model`에 데이터를 담아 `greet.jsp`로 전달하고, 최종적으로 "안녕하세요, 홍길동님!"과 메시지가 포함된 HTML 페이지가 렌더링됩니다.
