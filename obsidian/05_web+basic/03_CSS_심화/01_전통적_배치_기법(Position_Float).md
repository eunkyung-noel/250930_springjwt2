---
tags:
  - css
  - layout
  - position
  - float
  - clearfix
---

# 01. 전통적 배치 기법 (Position, Float)

Flexbox와 Grid가 등장하기 전, 웹 레이아웃은 주로 `float`과 `position` 속성을 사용하여 구현되었습니다. 이 기법들은 여전히 특정 상황에서 유용하지만, 복잡한 반응형 레이아웃을 만들기에는 한계가 명확했습니다.

#학습목표

- `float`을 이용해 요소를 좌우로 배치하고, `clearfix`의 필요성을 이해합니다.
- `position` 속성의 값(`static`, `relative`, `absolute`, `fixed`, `sticky`)들의 차이를 이해하고 활용할 수 있습니다.
- 전통적 배치 기법의 한계를 설명하고, 왜 최신 CSS 레이아웃 기술이 필요한지 이해합니다.

---

## 1. `float` 속성

#float #clearfix

`float` 속성은 본래 신문이나 잡지처럼 텍스트가 이미지를 감싸 흐르도록 하기 위해 고안되었습니다. `float: left;` 또는 `float: right;`를 사용하면 해당 요소는 보통의 문서 흐름에서 벗어나 왼쪽이나 오른쪽으로 '떠다니게' 됩니다.

### 사용 예시: 이미지와 텍스트 배치

```html
<style>
  .news-image {
    float: left;
    width: 150px;
    margin-right: 15px;
  }
</style>

<article>
  <img src="image.jpg" alt="뉴스 이미지" class="news-image" />
  <p>
    이 텍스트는 이미지를 감싸며 흐릅니다. float 속성은 원래 이런 용도로
    설계되었습니다. 텍스트가 길어지면 이미지의 오른쪽과 아래쪽으로 자연스럽게
    이어집니다.
  </p>
</article>
```

### `float`의 한계: 부모 높이 붕괴 (Height Collapse)

`float`된 요소는 문서의 일반적인 흐름(normal flow)에서 벗어나기 때문에, 부모 요소는 `float`된 자식의 높이를 인식하지 못합니다. 이로 인해 부모 요소의 높이가 0으로 계산되어 레이아웃이 깨지는 '높이 붕괴' 현상이 발생합니다.

```html
<style>
  .parent {
    border: 2px solid red;
  }
  .child {
    float: left;
    width: 100px;
    height: 100px;
    background: lightblue;
  }
</style>

<div class="parent">
  <div class="child">Float 자식</div>
  <!-- 부모 요소의 빨간 테두리가 자식을 감싸지 못함 -->
</div>
```

#### 해결책: `clearfix` 기법

이 문제를 해결하기 위해 `clearfix`라는 핵(hack) 기법이 사용되었습니다. `float`된 요소들 다음에 오는 가상 요소(`::after`)를 만들어 `clear: both;` 속성을 적용하는 방식입니다.

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

```html
<div class="parent clearfix">
  <div class="child">Float 자식</div>
</div>
```

`float`은 다단 컬럼 레이아웃에도 사용되었지만, 순서 변경이 어렵고 `clearfix`를 계속 신경 써야 하는 등 복잡한 레이아웃에는 부적합했습니다.

---

## 2. `position` 속성

#position

`position` 속성은 문서 흐름을 기준으로 요소를 원하는 위치에 배치하는 데 사용됩니다. `top`, `right`, `bottom`, `left` 속성과 함께 쓰입니다.

### `position`의 주요 값

| 값         | 기준                                                 | 특징                                                                                                       |
| :--------- | :--------------------------------------------------- | :--------------------------------------------------------------------------------------------------------- |
| `static`   | (기본값)                                             | 문서의 일반적인 흐름에 따라 배치됩니다. `top`, `left` 등 오프셋 속성이 적용되지 않습니다.                  |
| `relative` | 요소 자신의 원래 위치                                | 일반적인 흐름은 유지하되, `top`, `left` 등으로 **원래 위치를 기준**으로 약간의 위치 조정을 할 수 있습니다. |
| `absolute` | 가장 가까운 `position: relative/absolute/fixed` 부모 | 일반적인 흐름에서 완전히 벗어납니다. 지정된 부모 요소를 기준으로 좌표를 통해 배치됩니다.                   |
| `fixed`    | 뷰포트 (브라우저 창)                                 | 일반적인 흐름에서 벗어나며, **화면의 특정 위치에 고정**됩니다. 스크롤해도 움직이지 않습니다.               |
| `sticky`   | 스크롤 위치                                          | 평소에는 `static`처럼 흐름을 따르다가, 스크롤이 특정 지점에 도달하면 `fixed`처럼 동작합니다.               |

### `relative`와 `absolute`의 관계

`position: absolute;`는 레이어(layer)를 띄우는 것과 같습니다. 이 때 기준점이 될 부모 요소에 `position: relative;`를 지정하는 것이 가장 일반적인 패턴입니다.

```html
<style>
  .card {
    position: relative; /* absolute 자식의 기준점 */
    width: 200px;
    height: 200px;
    border: 1px solid gray;
  }
  .badge {
    position: absolute;
    top: 10px;
    right: 10px;
    background: red;
    color: white;
    padding: 5px;
  }
</style>

<div class="card">
  카드 콘텐츠
  <span class="badge">New</span>
</div>
```

`.badge`는 `.card`의 오른쪽 상단 모서리를 기준으로 배치됩니다. 만약 `.card`에 `position: relative;`가 없다면, `.badge`는 body나 다른 상위 요소를 기준으로 위치하게 되어 의도와 다른 결과를 낳습니다.

### `position`의 한계

- **복잡성**: 요소들이 문서 흐름을 벗어나기 시작하면 서로 겹치거나 반응형 디자인에 대응하기 어려워집니다.
- **콘텐츠 유연성 부족**: 요소의 크기가 고정되어 있지 않으면 정렬이나 배치가 까다롭습니다.

---

## 3. 전통적 기법의 문제점 요약

- **유지보수의 어려움**: `float`과 `clearfix`, `position` 속성들이 복잡하게 얽히면 코드를 이해하고 수정하기 어렵습니다.
- **반응형 대응의 한계**: 화면 크기에 따라 요소의 순서를 바꾸거나 유연하게 크기를 조절하는 것이 매우 복잡합니다.
- **수직 정렬의 부재**: 요소를 수직 중앙에 배치하는 간단하고 신뢰할 수 있는 방법이 없었습니다.

이러한 문제들을 해결하기 위해 **Flexbox**와 **Grid**라는 현대적인 CSS 레이아T웃 모듈이 등장했습니다. 이들은 복잡한 레이아웃 요구사항을 훨씬 간단하고 논리적인 코드로 해결할 수 있게 해줍니다.
