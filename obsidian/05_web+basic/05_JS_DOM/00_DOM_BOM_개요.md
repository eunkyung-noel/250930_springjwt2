# 00. DOM과 BOM 개요

#dom #document-object-model #돔 #bom #browser-object-model #봄

JavaScript가 웹 브라우저 환경에서 동적으로 페이지를 제어하는 핵심 기술은 DOM과 BOM입니다. 이 둘은 종종 함께 사용되지만, 명확히 다른 역할을 수행합니다.

---

## 1. DOM (Document Object Model)

#dom #document-object-model #돔

**DOM은 웹 페이지(HTML 문서)의 구조화된 표현**입니다. 브라우저는 HTML 코드를 읽어들여, 각 태그와 텍스트를 '노드(Node)'라는 객체로 변환하고 이들의 관계를 트리 구조로 구성합니다. JavaScript는 `document`라는 전역 객체를 통해 이 DOM 트리에 접근하여 페이지의 구조, 내용, 스타일을 동적으로 변경할 수 있습니다.

```mermaid
graph TD
    A[Document] --> B[html];
    B --> C[head];
    B --> D[body];
    C --> E[title];
    E --> F[텍스트 노드: "My Page"];
    D --> G[h1 id="title"];
    G --> H[텍스트 노드: "Welcome"];
    D --> I[p class="content"];
    I --> J[텍스트 노드: "Hello, World!"];
```

- **주요 역할**: HTML 문서의 모든 요소(Element)와 내용(Content)을 제어합니다.
- **특징**: W3C 표준으로, 모든 브라우저에서 동일한 방식으로 동작하는 것을 목표로 합니다.
- **핵심 객체**: `document`

---

## 2. BOM (Browser Object Model)

#bom #browser-object-model #봄

**BOM은 브라우저의 창이나 프레임 등, 웹 페이지의 내용을 제외한 브라우저의 모든 것을 제어**하기 위한 객체 모델입니다. 여기에는 새 창을 열거나, URL을 변경하거나, 화면 크기 정보를 얻는 등의 기능이 포함됩니다.

BOM은 표준화되어 있지 않아 브라우저 제조사마다 일부 차이가 있을 수 있지만, 대부분의 공통적인 기능(예: `window`, `location`, `navigator`)은 거의 모든 브라우저에서 지원됩니다.

- **주요 역할**: 브라우저 탭, URL, 화면 정보, 저장소 등 브라우저 환경 자체를 제어합니다.
- **특징**: 표준화가 미비하여 브라우저별로 구현이 다를 수 있습니다.
- **핵심 객체**: `window` (JavaScript의 최상위 전역 객체)

---

## 3. `window`와 `document`의 관계

`window` 객체는 BOM의 핵심이자 JavaScript의 전역 객체입니다. `document` 객체는 `window` 객체의 속성 중 하나로, 현재 창에 로드된 문서를 가리킵니다.

```mermaid
graph LR
    A[window (BOM, 전역 객체)] --> B{document (DOM)};
    A --> C{location};
    A --> D{navigator};
    A --> E{history};
    A --> F{screen};
    A --> G{localStorage};

    subgraph "BOM API"
        C; D; E; F; G;
    end

    subgraph "DOM API"
        B
    end
```

- `window.document`는 `document`와 같습니다.
- `alert()`, `setTimeout()` 같은 전역 함수들도 사실은 `window` 객체의 메서드입니다. (`window.alert()`)

결론적으로, **DOM은 문서(`document`)를, BOM은 브라우저(`window`)를 다루는 API**라고 이해할 수 있습니다.
