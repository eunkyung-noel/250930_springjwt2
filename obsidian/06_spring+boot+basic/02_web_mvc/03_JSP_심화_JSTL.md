# JSP 심화: JSTL, EL, 페이지 전환

#JSP #JSTL #EL #표현언어 #ExpressionLanguage #Forward #Redirect #PRG패턴

JSP를 효과적으로 사용하기 위해서는 스크립틀릿(`<% ... %>`)의 사용을 최소화하고, **EL(Expression Language)**과 **JSTL(JSP Standard Tag Library)**을 적극적으로 활용하는 것이 중요합니다. 또한, 웹 애플리케이션의 흐름을 제어하는 `Forward`와 `Redirect`의 차이를 이해하는 것이 필수적입니다.

---

## 1. EL (Expression Language)

EL은 JSP에서 데이터를 더 간결하고 편리하게 표현하기 위한 언어입니다. `${표현식}` 구문을 사용하여 `Model`에 담긴 속성 값이나 객체의 프로퍼티에 쉽게 접근할 수 있습니다.

- **주요 기능**:
  - `Model` 속성 접근: `${attributeName}`
  - 객체 프로퍼티 접근: `${user.name}` (내부적으로 `user.getName()` 호출)
  - 산술, 비교, 논리 연산: `${price * 1.1}`, `${count > 10}`, `${empty userList}`
- **장점**: 스크립틀릿 표현식(`<%= ... %>`)보다 코드가 깔끔하고, `null` 값에 대한 처리가 더 안전합니다(오류 대신 공백 출력).

---

## 2. JSTL (Jakarta Standard Tag Library)

JSTL은 JSP에서 자주 사용되는 로직(조건문, 반복문, 포맷팅 등)을 스크립틀릿 없이 태그 형태로 사용할 수 있게 만든 표준 라이브러리입니다. JSTL을 사용하면 뷰(View)의 코드를 훨씬 더 깔끔하고 가독성 높게 유지할 수 있습니다.

### `build.gradle`에 JSTL 의존성 추가

JSTL을 사용하려면 `build.gradle`에 관련 라이브러리를 추가해야 합니다.

```groovy
// build.gradle
dependencies {
    // ... 다른 의존성

    // JSTL 라이브러리 (Jakarta EE 10 호환)
    implementation 'jakarta.servlet.jsp.jstl:jakarta.servlet.jsp.jstl-api'
    implementation 'org.glassfish.web:jakarta.servlet.jsp.jstl'
}
```

### JSP 파일에 JSTL 사용 선언

JSP 파일 상단에 `<%@ taglib ... %>` 지시어를 사용하여 사용할 라이브러리를 선언합니다. `prefix`는 태그를 사용할 때의 접두사(보통 `c`를 사용)를, `uri`는 라이브러리의 식별자를 지정합니다.

**⚠️ Spring Boot 3 (Jakarta EE) 부터 `uri` 주소가 변경되었습니다.**

- **구버전 (Java EE)**: `http://java.sun.com/jsp/jstl/core`
- **신버전 (Jakarta EE)**: `jakarta.tags.core`

```jsp
<%-- /WEB-INF/views/example.jsp --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%-- JSTL core 라이브러리 사용 선언 (신버전) --%>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
```

### 주요 Core 태그 사용법

- **`<c:if>`**: 조건이 참일 때만 내부 콘텐츠를 실행합니다.
- **`<c:forEach>`**: 컬렉션(List, Map 등)의 각 요소를 순회합니다.

#### 예제: `ExampleController.java` 및 `example.jsp`

```java
// ExampleController.java
@Controller
public class ExampleController {
    @GetMapping("/example")
    public String elJstlExample(Model model) {
        List<String> members = List.of("김철수", "이영희", "박민준");
        model.addAttribute("memberList", members);
        model.addAttribute("isManager", true);
        return "example";
    }
}
```

```jsp
<%-- /WEB-INF/views/example.jsp --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<html>
<head>
    <title>EL & JSTL 예제</title>
</head>
<body>
    <%-- c:if를 사용하여 조건부 렌더링 --%>
    <c:if test="${isManager}">
        <p>[관리자 모드]</p>
    </c:if>

    <h3>회원 목록</h3>
    <ul>
        <%-- c:forEach를 사용하여 리스트 순회 --%>
        <c:forEach items="${memberList}" var="name">
            <li>${name}</li>
        </c:forEach>
    </ul>

    <%-- 리스트가 비어있을 경우 처리 --%>
    <c:if test="${empty memberList}">
        <p>등록된 회원이 없습니다.</p>
    </c:if>
</body>
</html>
```

---

## 3. Forward vs. Redirect

페이지 전환을 제어하는 두 가지 주요 방식입니다.

| 구분             | Forward                                 | Redirect                                |
| ---------------- | --------------------------------------- | --------------------------------------- |
| **주체**         | 서버 (WAS)                              | 클라이언트 (브라우저)                   |
| **URL 변경**     | 변경되지 않음                           | 변경됨                                  |
| **Request 객체** | 동일한 요청이 유지됨 (데이터 공유 가능) | 새로운 요청이 생성됨 (데이터 공유 불가) |
| **요청 횟수**    | 1회                                     | 2회 (최초 요청 → 302 응답 → 재요청)     |
| **주요 용도**    | 단순 조회, 서버 내부 흐름 제어          | 데이터 변경(C/U/D) 후 화면 전환         |
| **Spring 반환**  | `return "forward:/some/url";`           | `return "redirect:/some/url";`          |

### PRG (Post-Redirect-Get) 패턴

`Redirect`는 **PRG 패턴**의 핵심입니다. 사용자가 폼을 제출하여 데이터 변경(POST)이 발생한 후, 서버는 바로 뷰를 렌더링하는 대신 클라이언트에게 다른 페이지로 이동하라는 `Redirect` 응답(302)을 보냅니다. 클라이언트는 이 응답을 받고 해당 페이지를 다시 `GET` 방식으로 요청합니다.

이 패턴을 사용하면 사용자가 데이터 처리 완료 페이지에서 **새로고침을 눌러도 이전의 POST 요청이 중복으로 제출되는 것을 방지**할 수 있습니다.

```java
// MemberController.java
@PostMapping("/add")
public String add(@ModelAttribute Member member) {
    memberRepository.add(member); // 데이터 저장 (POST)

    // 데이터 처리 후에는 반드시 Redirect를 사용하여 중복 등록을 방지 (PRG 패턴)
    // /members URL로 다시 요청하라는 응답을 브라우저에 보냄 (Redirect)
    return "redirect:/members";
}

@GetMapping("/members")
public String list(Model model) { // (GET)
    model.addAttribute("members", memberRepository);
    return "member/list";
}
```
