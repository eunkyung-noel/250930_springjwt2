# 01. DOM 노드 탐색과 조작

#dom #노드탐색 #노드조작 #node-selection #node-manipulation

JavaScript로 웹 페이지를 동적으로 만들기 위한 첫 단계는 원하는 HTML 요소를 정확히 **탐색(선택)**하고, 그 요소의 내용, 속성, 스타일을 **조작**하는 것입니다.

---

## 1. DOM 노드 탐색 (선택)

JavaScript가 HTML 요소를 제어하려면 먼저 문서(document)에서 해당 요소를 찾아야 합니다. 이때 사용되는 주요 메서드는 다음과 같습니다.

| 메서드                                  | 설명                                                                       | 반환 값                |
| --------------------------------------- | -------------------------------------------------------------------------- | ---------------------- |
| `document.getElementById('id')`         | 주어진 `id` 속성을 가진 **단일** 요소를 찾습니다. 가장 빠르고 명확합니다.  | 요소 객체 또는 `null`  |
| `document.querySelector('selector')`    | 주어진 CSS 선택자에 해당하는 **첫 번째** 요소를 찾습니다. 매우 유연합니다. | 요소 객체 또는 `null`  |
| `document.querySelectorAll('selector')` | 주어진 CSS 선택자에 해당하는 **모든** 요소를 찾습니다.                     | `NodeList` (유사 배열) |

### 예제 코드

```html
<!-- index.html -->
<div id="main-container">
  <h1 id="title">고유한 제목</h1>
  <p class="content">첫 번째 단락.</p>
  <p class="content">두 번째 단락.</p>
</div>
```

```javascript
// app.js

// 1. ID로 선택하기
const titleElement = document.getElementById("title");
console.log(titleElement); // <h1 id="title">...</h1>

// 2. CSS 선택자로 첫 번째 요소 선택하기
const firstContent = document.querySelector(".content");
console.log(firstContent); // <p class="content">첫 번째 단락.</p>

// 3. CSS 선택자로 모든 요소 선택하기
const allContents = document.querySelectorAll(".content");
console.log(allContents); // NodeList [ <p.content>, <p.content> ]

// NodeList는 forEach를 사용할 수 있습니다.
allContents.forEach((p) => {
  console.log(p.textContent);
});
```

---

## 2. 내용 및 속성 조작

요소를 선택했다면, 그 내용을 바꾸거나 HTML 속성을 변경할 수 있습니다.

### 가. 내용(Content) 변경: `textContent` vs `innerHTML`

| API           | 특징                                                                               | 주요 용도                                   | 보안                |
| :------------ | :--------------------------------------------------------------------------------- | :------------------------------------------ | :------------------ |
| `textContent` | **순수 텍스트**만 다룹니다. HTML 태그를 문자열 그대로 취급합니다.                  | **안전한 텍스트 삽입** (사용자 입력 등)     | **안전 (XSS 방지)** |
| `innerHTML`   | **HTML 코드로 해석**하여 렌더링합니다.                                             | **동적 마크업** 삽입 (개발자가 작성한 코드) | **주의 (XSS 위험)** |
| `innerText`   | 사용자에게 **보이는(렌더링된)** 텍스트만 다룹니다. `display:none` 등은 제외됩니다. | 화면에 보이는 텍스트 추출                   | 안전                |

> **보안 경고 (XSS)**: `innerHTML`에 사용자가 입력한 값을 그대로 넣으면, 악의적인 `<script>` 태그가 실행되어 사이트가 공격당할 수 있습니다(Cross-Site Scripting). **사용자 입력값은 반드시 `textContent`를 사용하세요.**

```javascript
const el = document.querySelector("#app");
const userInput = '<img src="x" onerror="alert(\'XSS 공격!\')">';

// ❌ 위험한 사용법
// el.innerHTML = userInput; // 경고창이 실행됨!

// ✅ 안전한 사용법
el.textContent = userInput; // '<img...>' 문자열이 그대로 화면에 보임
```

### 나. 속성(Attribute) 변경

`setAttribute`, `getAttribute`를 사용하거나 `dataset` 프로퍼티를 통해 `data-*` 속성을 쉽게 제어할 수 있습니다.

```javascript
const link = document.querySelector("#my-link");

// 속성 설정
link.setAttribute("href", "https://www.google.com");
link.setAttribute("target", "_blank");

// 속성 읽기
const href = link.getAttribute("href");
console.log(href); // https://www.google.com

// data-* 속성 제어
link.dataset.linkId = "12345"; // data-link-id="12345"로 설정됨
console.log(link.dataset.linkId); // '12345'
```

---

## 3. 클래스 및 스타일 조작

### 가. 클래스(Class) 조작: `classList`

`classList` 프로퍼티는 요소의 클래스를 제어하는 유용한 메서드들을 제공합니다.

- `element.classList.add('className')`: 클래스 추가
- `element.classList.remove('className')`: 클래스 제거
- `element.classList.toggle('className')`: 클래스가 있으면 제거, 없으면 추가
- `element.classList.contains('className')`: 클래스 포함 여부 확인 (true/false)

```javascript
const box = document.querySelector(".box");
box.classList.add("active"); // 'box active'
box.classList.remove("box"); // 'active'
box.classList.toggle("visible"); // 'active visible'
console.log(box.classList.contains("active")); // true
```

### 나. 스타일(Style) 조작

`style` 프로퍼티를 통해 요소의 인라인 스타일을 직접 변경할 수 있습니다.

- **주의**: CSS 속성명이 하이픈(-)을 포함하면(예: `background-color`), 카멜 케이스(camelCase)로 변환해야 합니다(예: `backgroundColor`).

```javascript
const title = document.getElementById("title");

// 개별 스타일 변경
title.style.color = "blue";
title.style.backgroundColor = "#f0f0f0";
title.style.fontSize = "24px";

// 여러 스타일 한번에 변경 (cssText)
title.style.cssText =
  "color: red; font-weight: bold; border-bottom: 2px solid red;";
```
