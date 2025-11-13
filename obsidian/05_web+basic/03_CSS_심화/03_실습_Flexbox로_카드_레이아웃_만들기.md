---
tags:
  - css
  - flexbox
  - layout
  - card-ui
  - practical-example
---

# 04. 실습: Flexbox로 카드 레이아웃 만들기

지금까지 배운 Flexbox 속성을 활용하여 웹사이트에서 흔히 볼 수 있는 '카드(Card)' UI 컴포넌트를 만들어 보겠습니다. 이 실습을 통해 Flexbox가 실제 프로젝트에서 어떻게 사용되는지 감을 잡을 수 있습니다.

#학습목표

- Flexbox를 사용하여 수평으로 여러 개의 카드를 나열하고, 반응형으로 줄바꿈되도록 할 수 있습니다.
- 카드 내부의 콘텐츠(이미지, 텍스트, 버튼)를 Flexbox를 사용하여 수직으로 정렬하고 공간을 분배할 수 있습니다.
- `flex-grow`를 활용하여 특정 콘텐츠 영역이 남은 공간을 모두 차지하도록 만들 수 있습니다.

---

## 1. 최종 목표

아래와 같은 반응형 카드 그리드를 만드는 것이 목표입니다.

- 여러 개의 카드가 나란히 배치됩니다.
- 화면 너비가 좁아지면 카드가 자동으로 다음 줄로 넘어갑니다.
- 각 카드 내부의 콘텐츠는 깔끔하게 정렬되어 있습니다. (이미지-텍스트-버튼 순)

```mermaid
graph TD
    subgraph Browser
        subgraph Card Container (display: flex, flex-wrap: wrap)
            direction LR
            A[Card 1] -- gap --> B[Card 2] -- gap --> C[Card 3]
        end
    end

    subgraph Card 1 (display: flex, flex-direction: column)
        direction TB
        D[Image] --> E[Text Content] --> F[Button]
    end

    style A fill:#fff,stroke:#ccc,stroke-width:2px
    style B fill:#fff,stroke:#ccc,stroke-width:2px
    style C fill:#fff,stroke:#ccc,stroke-width:2px
    style D fill:#eee,stroke:#999
    style E fill:#fff,stroke:#fff
    style F fill:skyblue,stroke:blue,color:#fff
```

---

## 2. HTML 구조

먼저 카드 레이아웃을 위한 기본 HTML 구조를 작성합니다. 카드들을 감싸는 `.card-container`와 개별 카드인 `.card`로 구성됩니다.

**`index.html`**

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Flexbox 카드 레이아웃 실습</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <h1>인기 여행지</h1>

    <div class="card-container">
      <article class="card">
        <img
          src="https://via.placeholder.com/400x250.png?text=Travel+Image+1"
          alt="여행지 이미지 1"
        />
        <div class="card-content">
          <h2>파리, 프랑스</h2>
          <p>
            에펠탑과 루브르 박물관이 있는 예술과 낭만의 도시입니다. 센 강변을
            거닐며 여유를 느껴보세요.
          </p>
          <a href="#" class="button">더 알아보기</a>
        </div>
      </article>

      <article class="card">
        <img
          src="https://via.placeholder.com/400x250.png?text=Travel+Image+2"
          alt="여행지 이미지 2"
        />
        <div class="card-content">
          <h2>교토, 일본</h2>
          <p>
            전통적인 사원과 아름다운 정원이 가득한 고즈넉한 도시입니다. 기모노를
            입고 옛 거리를 산책해 보세요.
          </p>
          <a href="#" class="button">더 알아보기</a>
        </div>
      </article>

      <article class="card">
        <img
          src="https://via.placeholder.com/400x250.png?text=Travel+Image+3"
          alt="여행지 이미지 3"
        />
        <div class="card-content">
          <h2>뉴욕, 미국</h2>
          <p>
            타임스 스퀘어의 화려함과 센트럴 파크의 평화로움이 공존하는 세계
            경제와 문화의 중심지입니다.
          </p>
          <a href="#" class="button">더 알아보기</a>
        </div>
      </article>
    </div>
  </body>
</html>
```

---

## 3. CSS 스타일링

이제 Flexbox를 사용하여 레이아웃을 잡아보겠습니다.

**`style.css`**

### 1단계: 기본 스타일 및 카드 컨테이너 설정

카드들을 감싸는 `.card-container`에 `display: flex`를 적용하여 카드들을 가로로 배치합니다. `flex-wrap: wrap`으로 화면이 좁아질 때 카드가 다음 줄로 넘어가도록 설정합니다.

```css
/* 기본 스타일 */
body {
  font-family: sans-serif;
  margin: 20px;
  background-color: #f4f4f4;
}

h1 {
  text-align: center;
  margin-bottom: 40px;
}

/* 카드 컨테이너 */
.card-container {
  display: flex;
  flex-wrap: wrap; /* 카드가 다음 줄로 넘어가도록 설정 */
  justify-content: center; /* 카드들을 중앙에 배치 */
  gap: 20px; /* 카드 사이의 간격 */
}
```

### 2단계: 개별 카드 기본 스타일

각 `.card`의 너비와 모양을 정의합니다. `flex: 1 1 300px;`는 카드의 기본 너비를 300px로 하되, 공간이 남으면 늘어나고 부족하면 줄어들도록 하여 유연한 반응형 동작을 만듭니다.

```css
/* 개별 카드 */
.card {
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  overflow: hidden; /* border-radius가 이미지에 적용되도록 */

  /* Flex 아이템 설정 */
  flex: 1 1 300px; /* grow, shrink, basis */
  max-width: 350px; /* 카드가 너무 커지는 것을 방지 */
}

.card img {
  width: 100%;
  display: block; /* 이미지 아래의 불필요한 여백 제거 */
}
```

### 3단계: 카드 내부 콘텐츠 정렬

카드 내부의 `.card-content` 영역을 Flex 컨테이너로 만들고, `flex-direction: column`을 적용하여 콘텐츠(제목, 문단, 버튼)를 수직으로 배치합니다.

```css
.card-content {
  padding: 20px;

  /* 내부 콘텐츠를 위한 Flex 컨테이너 */
  display: flex;
  flex-direction: column;
  height: 100%; /* 부모(.card)의 높이를 채우도록 설정 */
}

.card-content h2 {
  margin-top: 0;
}

.card-content p {
  /* p 태그가 남은 공간을 모두 차지하도록 설정 */
  flex-grow: 1;
  margin-bottom: 20px;
}

.button {
  display: block;
  background-color: #007bff;
  color: white;
  text-align: center;
  padding: 10px;
  text-decoration: none;
  border-radius: 5px;
  margin-top: auto; /* 버튼을 항상 맨 아래에 위치시킴 */
}
```

- **`flex-grow: 1;`** on `p`: 문단(`p`)이 카드 내용에서 제목과 버튼을 제외한 **모든 남은 수직 공간을 차지**하게 만듭니다. 이 덕분에 카드들의 높이가 달라도 버튼은 항상 같은 하단 라인에 정렬되는 것처럼 보입니다.
- **`margin-top: auto;`** on `.button`: `flex-grow` 대신 버튼에 `margin-top: auto`를 적용해도 비슷한 효과를 낼 수 있습니다. 버튼 위쪽의 모든 여백을 자동으로 채워 버튼을 아래로 밀어냅니다.

---

## 4. 결과 확인

HTML 파일을 브라우저에서 열고 창의 너비를 조절해 보세요. 카드들이 화면 크기에 맞춰 유연하게 재배치되는 것을 확인할 수 있습니다. 각 카드 내부의 버튼들도 항상 하단에 정렬되어 깔끔한 UI를 유지합니다.

이처럼 Flexbox는 컴포넌트 단위의 레이아웃을 만들 때 매우 직관적이고 강력한 도구입니다.
