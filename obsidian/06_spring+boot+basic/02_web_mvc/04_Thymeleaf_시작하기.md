# Thymeleaf 시작하기

#Thymeleaf #타임리프 #템플릿엔진 #TemplateEngine #서버사이드렌더링 #SSR

Thymeleaf는 현대적인 서버 사이드 템플릿 엔진으로, Spring Boot에서 공식적으로 권장하는 뷰 기술입니다. JSP의 단점을 보완하고, 순수 HTML에 가까운 '내추럴 템플릿'을 지향하여 개발자와 디자이너 간의 협업을 용이하게 만듭니다.

---

## 1. Thymeleaf의 특징과 장점

- **내추럴 템플릿 (Natural Templates)**
  - Thymeleaf 템플릿은 웹 브라우저에서 직접 열어도 깨지지 않는 순수한 HTML 구조를 유지합니다. `th:*` 속성은 일반 HTML 속성처럼 동작하므로, 서버를 실행하지 않고도 UI 디자인을 확인할 수 있습니다.
- **JAR 패키징과의 완벽한 호환**
  - Spring Boot의 기본 패키징 방식인 JAR와 완벽하게 호환됩니다. JSP처럼 WAR 패키징이나 외부 WAS 설정이 필요 없어, 내장 서버만으로 간편하게 애플리케이션을 실행할 수 있습니다.
- **간결하고 강력한 문법**
  - `th:text`, `th:if`, `th:each` 등 직관적인 속성을 사용하여 데이터를 표현하고, 조건문, 반복문 등을 쉽게 처리할 수 있습니다.
- **레이아웃 관리 기능**
  - 공통된 페이지 요소(헤더, 푸터 등)를 조각(Fragment)으로 만들어 재사용할 수 있는 강력한 레이아웃 기능을 제공하여 코드 중복을 줄이고 유지보수성을 높입니다.

---

## 2. `build.gradle` 설정

Thymeleaf를 사용하기 위한 설정은 매우 간단합니다. `build.gradle` 파일에 `spring-boot-starter-thymeleaf` 의존성만 추가하면 됩니다.

```groovy
// build.gradle
dependencies {
    // Spring Web MVC, 내장 톰캣 등 웹 개발 필수 라이브러리 포함
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // Thymeleaf 템플릿 엔진 스타터
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

이 스타터 라이브러리를 추가하면, Spring Boot는 자동으로 Thymeleaf 관련 설정을 구성합니다.

---

## 3. 자동 설정 및 파일 위치

`spring-boot-starter-thymeleaf`를 추가하면 다음과 같은 설정이 자동으로 적용됩니다.

- **뷰 리졸버(ViewResolver) 자동 설정**:
  - `application.properties`에 JSP처럼 `prefix`나 `suffix`를 설정할 필요가 없습니다.
  - 기본적으로 `src/main/resources/templates/` 경로에 있는 `.html` 파일을 뷰로 인식합니다.
- **파일 위치**:
  - Thymeleaf 템플릿 파일은 `src/main/resources/templates/` 디렉터리 아래에 위치해야 합니다.

### 디렉터리 구조 예시

```
src
└── main
    ├── java
    │   └── com/example/controller
    │       └── HomeController.java
    └── resources
        ├── static
        │   └── css/style.css
        └── templates
            └── home.html
```

---

## 4. 간단한 예제

### `HomeController.java`

```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Welcome to Thymeleaf!");
        // ViewResolver가 /resources/templates/home.html 파일을 찾아서 렌더링
        return "home";
    }
}
```

### `home.html`

Thymeleaf 템플릿을 사용하려면 `<html>` 태그에 `xmlns:th="http://www.thymeleaf.org"` 속성을 선언해야 합니다.

```html
<!-- /src/main/resources/templates/home.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8" />
    <title>Home</title>
  </head>
  <body>
    <h1>Hello, Thymeleaf!</h1>

    <%-- p 태그의 텍스트를 모델에서 받은 message 값으로 대체합니다. th:text
    속성이 없다면, 서버 없이 파일을 열었을 때 "Default Message"가 보입니다.
    이것이 바로 '내추럴 템플릿'의 장점입니다. --%>
    <p th:text="${message}">Default Message</p>
  </body>
</html>
```

이제 애플리케이션을 실행하고 `http://localhost:8080`에 접속하면, "Welcome to Thymeleaf!"라는 메시지가 화면에 출력되는 것을 확인할 수 있습니다.
