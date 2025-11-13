---
tags:
  - css
  - display
  - block
  - inline
  - inline-block
---

# 00. CSS Display 모델

CSS에서 모든 HTML 요소는 사각형 박스로 렌더링됩니다. `display` 속성은 이 박스가 렌더링되는 방식을 결정하는 가장 기본적인 CSS 속성 중 하나입니다. 요소가 화면에서 어떻게 보이고, 다른 요소와 어떻게 상호작용할지를 정의합니다.

#학습목표

- `display` 속성의 역할을 이해합니다.
- `block`, `inline`, `inline-block`의 차이점과 각각의 특징을 설명할 수 있습니다.
- 각 `display` 값이 어떤 상황에 사용되는지 이해합니다.

---

## 1. `display` 속성의 주요 값

`display` 속성은 요소의 **외부 디스플레이 유형(outer display type)**과 **내부 디스플레이 유형(inner display type)**을 정의합니다. 여기서는 가장 기본이 되는 외부 디스플레이 유형 3가지를 알아봅니다.

| 값 (`display`) | 외부 디스플레이 유형       | 특징                                                                                                                                   | 주요 용도                                                  |
| :------------- | :------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------- |
| `block`        | 블록 (Block)               | 한 줄 전체를 차지하며, 너비와 높이, 여백(`margin`)을 모두 지정할 수 있습니다.                                                          | 레이아웃의 큰 틀, 섹션, 문단 (`div`, `p`, `h1`, `section`) |
| `inline`       | 인라인 (Inline)            | 콘텐츠의 너비만큼만 공간을 차지하며, 줄바꿈 없이 다른 인라인 요소와 나란히 배치됩니다. 너비, 높이, 상하 `margin`을 지정할 수 없습니다. | 텍스트의 일부 강조, 링크 (`span`, `a`, `strong`, `em`)     |
| `inline-block` | 인라인-블록 (Inline-Block) | `inline`처럼 다른 요소와 나란히 배치되면서도, `block`처럼 너비, 높이, 여백을 모두 지정할 수 있습니다.                                  | 버튼, 배지, 작은 아이콘 등                                 |

---

## 2. `block` 요소

#display_block

`block` 요소는 이름 그대로 '덩어리'입니다.

- **새로운 줄에서 시작**하여 항상 한 줄 전체의 너비를 차지합니다. (`width: 100%`가 기본 동작)
- `width`, `height`, `margin`, `padding` 속성을 모두 사용하여 크기와 여백을 자유롭게 제어할 수 있습니다.
- 주로 웹 페이지의 구조적인 부분을 만들 때 사용됩니다. (e.g., 헤더, 푸터, 섹션, 기사 등)

```html
<div style="background: lightblue; border: 1px solid blue;">
  이것은 block 요소(div)입니다.
</div>
<p style="background: lightgreen; border: 1px solid green;">
  이것도 block 요소(p)입니다. 자동으로 줄바꿈됩니다.
</p>
```

---

## 3. `inline` 요소

#display_inline

`inline` 요소는 텍스트의 흐름(line) 안에 들어가는 요소입니다.

- **줄바꿈 없이** 다른 `inline` 요소들과 한 줄에 나란히 배치됩니다.
- 콘텐츠의 크기만큼만 너비를 차지합니다.
- **`width`와 `height` 속성을 적용할 수 없습니다.**
- **`margin-top`과 `margin-bottom` 속성을 적용할 수 없습니다.** (좌우 `margin-left`, `margin-right`는 가능)
- `padding`은 적용되지만, 주변 요소를 밀어내지 않고 시각적으로 겹칠 수 있습니다.

```html
<span style="background: lightcoral; border: 1px solid red;">
  이것은 inline 요소(span)입니다.
</span>
<a href="#" style="background: lightyellow; border: 1px solid orange;">
  이것도 inline 요소(a)이며, 옆에 붙습니다.
</a>
<strong>width와 height는 적용되지 않습니다.</strong>
```

---

## 4. `inline-block` 요소

#display_inline_block

`inline-block`은 `inline`과 `block`의 특징을 결합한 하이브리드 모델입니다.

- `inline`처럼 **줄바꿈 없이** 다른 요소 옆에 배치될 수 있습니다.
- `block`처럼 **`width`, `height`, `margin`, `padding`을 모두 적용**할 수 있습니다.
- `inline`의 배치 특성과 `block`의 크기 조절 능력이 모두 필요할 때 유용합니다. (e.g., 내비게이션 바의 메뉴 항목, 버튼)

```html
<div style="border: 1px solid gray; padding: 10px;">
  <button
    style="display: inline-block; width: 100px; height: 40px; background: skyblue;"
  >
    버튼 A
  </button>
  <span
    style="display: inline-block; width: 100px; height: 40px; background: lightgreen; text-align: center; line-height: 40px;"
  >
    박스 B
  </span>
  <div
    style="display: inline-block; width: 100px; height: 40px; background: lightpink;"
  >
    박스 C
  </div>
</div>
```

위 예제에서 `button`의 기본 `display` 값은 브라우저마다 다르지만 보통 `inline-block`과 유사하게 동작합니다. `span`과 `div`에 `display: inline-block`을 적용하여 너비와 높이를 가지면서도 한 줄에 배치되도록 만들었습니다.

이 세 가지 기본 `display` 모델을 이해하는 것은 CSS 레이아웃의 첫걸음입니다. 다음 문서에서는 이를 응용한 전통적인 배치 기법과 최신 Flexbox, Grid 레이아웃에 대해 알아봅니다.
