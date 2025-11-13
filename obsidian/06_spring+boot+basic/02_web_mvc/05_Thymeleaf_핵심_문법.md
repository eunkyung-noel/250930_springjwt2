# Thymeleaf 핵심 문법

#Thymeleaf #타임리프 #문법 #th_text #th_each #th_if #th_object #레이아웃

Thymeleaf는 HTML 태그에 `th:*` 속성을 추가하여 동적인 콘텐츠를 처리합니다. 여기서는 가장 자주 사용되는 핵심 문법과 레이아웃 관리 방법을 알아봅니다.

---

## 1. 주요 `th:*` 속성

### `th:text`와 `th:utext`

- `th:text`: 태그의 텍스트 콘텐츠를 지정된 값으로 변경합니다. HTML 태그를 문자로 처리하므로, XSS(Cross-Site Scripting) 공격을 방지하는 데 효과적입니다. (기본적으로 사용 권장)
- `th:utext`: 'Unescaped Text'의 약자로, HTML 태그를 이스케이프(escape)하지 않고 그대로 렌더링합니다. HTML 코드를 직접 출력해야 할 때 사용하지만, 신뢰할 수 없는 데이터에 사용할 경우 보안에 취약할 수 있습니다.

```html
<p th:text="${message}">기본 메시지</p>
<!-- ${message} 값이 "<b>Hello</b>" 라면, 화면에는 "<b>Hello</b>" 문자열이 그대로 보임 -->

<p th:utext="${message}">기본 메시지</p>
<!-- ${message} 값이 "<b>Hello</b>" 라면, 화면에는 굵은 글씨의 Hello가 보임 -->
```

### `th:if`와 `th:unless`

- `th:if`: 주어진 조건이 `true`일 경우에만 해당 태그를 렌더링합니다.
- `th:unless`: `th:if`와 반대로, 주어진 조건이 `false`일 경우에만 해당 태그를 렌더링합니다.

```html
<p th:if="${isManager}">[관리자 모드]</p>
<p th:unless="${isManager}">[일반 사용자 모드]</p>
```

### `th:each`

- JSTL의 `<c:forEach>`처럼 컬렉션(List, Map 등)을 순회하며 반복적으로 태그를 생성합니다.

```html
<ul>
  <li th:each="name : ${memberList}" th:text="${name}">회원 이름</li>
</ul>

<!-- memberList가 비어있을 경우, li 태그는 전혀 렌더링되지 않음 -->
```

---

## 2. Form 처리: `th:object`와 `th:field`

Thymeleaf는 `@ModelAttribute`를 사용한 폼 데이터 처리를 매우 직관적으로 만들어줍니다.

- **`th:object`**: `<form>` 태그에 사용하여 폼 데이터를 바인딩할 객체(DTO)를 지정합니다.
- **`th:field`**: `<input>`, `<select>` 등의 태그에 사용하여 `th:object`로 지정된 객체의 필드와 매핑합니다. `id`, `name`, `value` 속성을 자동으로 생성하여 코드 중복을 크게 줄여줍니다. `*{...}` 문법은 `th:object`로 지정된 객체의 필드에 접근하는 축약 표현입니다.

#### 예제: `MemberController.java` 및 `addForm.html`

```java
// MemberController.java
@Controller
@RequestMapping("/members")
public class MemberController {
    // ...
    @GetMapping("/add")
    public String addForm(Model model) {
        // th:object에서 사용할 수 있도록 빈 Member 객체를 모델에 담아 전달
        model.addAttribute("member", new Member());
        return "member/addForm";
    }

    @PostMapping("/add")
    public String add(@ModelAttribute Member member) {
        memberRepository.add(member);
        return "redirect:/members";
    }
}
```

```html
<!-- /resources/templates/member/addForm.html -->
<form action="/members/add" th:action th:object="${member}" method="post">
  <div>
    <label for="memberId">아이디:</label>
    <!-- th:field="*{memberId}"는 id="memberId", name="memberId", value=""를 자동으로 생성 -->
    <input type="text" th:field="*{memberId}" required />
  </div>
  <div>
    <label for="name">이름:</label>
    <input type="text" th:field="*{name}" required />
  </div>
  <button type="submit">등록</button>
</form>
```

---

## 3. 레이아웃 관리 (Fragment)

JSP의 `<jsp:include>`처럼 Thymeleaf도 공통 영역을 조각(Fragment)으로 분리하여 재사용할 수 있습니다.

1.  **조각(Fragment) 정의**: 공통으로 사용할 HTML 영역에 `th:fragment="이름"` 속성을 부여합니다.

    ```html
    <!-- /resources/templates/_layout/header.html -->
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
      <head th:fragment="header-fragment">
        <meta charset="UTF-8" />
        <title>회원 관리 시스템</title>
      </head>
    </html>
    ```

2.  **조각 포함**: `th:insert` 또는 `th:replace` 속성을 사용하여 다른 HTML 파일에서 조각을 가져옵니다. `~{...}`는 프래그먼트 표현식입니다.

    - `th:insert`: 해당 태그의 **내부**에 프래그먼트를 삽입합니다.
    - `th:replace`: 해당 태그 **자체**를 프래그먼트로 대체합니다.

    ```html
    <!-- /resources/templates/member/list.html -->
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
      <!-- head 태그 전체가 header.html의 header-fragment로 교체됨 -->
      <head th:replace="~{_layout/header :: header-fragment}"></head>

      <body>
        <!-- div 태그 내부에 gnb-fragment가 삽입됨 -->
        <div th:insert="~{_layout/header :: gnb-fragment}"></div>

        <h2>회원 목록</h2>
        <!-- ... -->
      </body>
    </html>
    ```

이러한 레이아웃 기능을 통해 헤더, 푸터, 내비게이션 바 등 공통 요소를 중앙에서 관리하여 일관성 있고 유지보수하기 쉬운 뷰를 구성할 수 있습니다.
