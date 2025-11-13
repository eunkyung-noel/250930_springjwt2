---
tags:
  - css
  - layout
  - flexbox
  - responsive-design
---

# 02. Flexbox 레이아웃

Flexbox(Flexible Box Layout)는 전통적인 레이아웃 방식(`float`, `position`)의 한계를 극복하고, 복잡한 웹 레이아웃을 쉽고 유연하게 구성하기 위해 설계된 **1차원 레이아웃 모델**입니다. 아이템들을 하나의 축(가로 또는 세로)을 기준으로 정렬하고, 공간을 분배하며, 순서를 제어하는 강력한 기능을 제공합니다.

#학습목표

- Flexbox의 핵심 개념인 컨테이너, 아이템, 주축(main axis), 교차축(cross axis)을 이해합니다.
- Flex 컨테이너에 적용하는 주요 속성(`flex-direction`, `justify-content`, `align-items` 등)을 사용하여 아이템을 정렬할 수 있습니다.
- Flex 아이템에 적용하는 속성(`flex-grow`, `flex-shrink`, `flex-basis`)을 사용하여 아이템의 크기를 유연하게 조절할 수 있습니다.

---

## 1. Flexbox의 핵심 개념

Flexbox 레이아웃은 **Flex 컨테이너(Container)**와 그 안의 **Flex 아이템(Item)**들로 구성됩니다.

- **Flex Container**: `display: flex;` 또는 `display: inline-flex;`가 적용된 부모 요소입니다.
- **Flex Item**: Flex 컨테이너의 직계 자식 요소들입니다.

### 주축(Main Axis)과 교차축(Cross Axis)

Flexbox는 두 개의 축을 기준으로 아이템을 배치합니다.

- **주축 (Main Axis)**: Flex 아이템들이 배치되는 기본 방향입니다. `flex-direction` 속성으로 방향을 정합니다. (기본값: `row` - 가로)
- **교차축 (Cross Axis)**: 주축에 수직인 축입니다.

```mermaid
graph TD
    subgraph Flex Container (flex-direction: row)
        direction LR
        A[Item 1] --> B[Item 2] --> C[Item 3]
        subgraph Main Axis (주축)
            A --> C
        end
    end
    subgraph Cross Axis (교차축)
        direction TB
        D --- E(( ))
    end

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#f9f,stroke:#333,stroke-width:2px
```

- `flex-direction: row` (기본값): 주축은 가로(왼쪽 → 오른쪽), 교차축은 세로(위 → 아래)
- `flex-direction: column`: 주축은 세로(위 → 아래), 교차축은 가로(왼쪽 → 오른쪽)

---

## 2. Flex 컨테이너 속성

컨테이너에 적용하여 아이템들의 전반적인 배치, 정렬, 간격을 제어합니다.

| 속성              | 설명                                                        | 주요 값                                                                             |
| :---------------- | :---------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| `display`         | Flexbox 컨텍스트를 생성합니다.                              | `flex`, `inline-flex`                                                               |
| `flex-direction`  | 주축의 방향을 설정합니다.                                   | `row` (기본), `column`, `row-reverse`, `column-reverse`                             |
| `flex-wrap`       | 아이템이 한 줄에 들어가지 않을 때 줄바꿈 여부를 결정합니다. | `nowrap` (기본), `wrap`, `wrap-reverse`                                             |
| `justify-content` | **주축** 방향으로 아이템들을 정렬합니다.                    | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items`     | **교차축** 방향으로 아이템들을 정렬합니다.                  | `stretch` (기본), `flex-start`, `flex-end`, `center`, `baseline`                    |
| `gap`             | 아이템 사이의 간격을 설정합니다.                            | `10px`, `1rem 2rem` (행 간격, 열 간격)                                              |

### `justify-content` 와 `align-items` 예시

```html
<style>
  .flex-container {
    display: flex;
    height: 150px;
    background: lightyellow;
    border: 2px solid navy;
    /* 아래 속성들을 바꿔보세요! */
    justify-content: center;
    align-items: center;
    gap: 10px;
  }
  .item {
    background: lightpink;
    padding: 20px;
    border: 1px solid deeppink;
  }
</style>

<div class="flex-container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

- `justify-content: space-between;`: 첫 아이템은 시작점, 마지막 아이템은 끝점에 붙고 나머지는 균등한 간격으로 배치됩니다.
- `align-items: center;`: 아이템들이 교차축의 중앙에 배치됩니다.

---

## 3. Flex 아이템 속성

개별 아이템에 적용하여 크기, 순서, 개별 정렬을 제어합니다.

| 속성          | 설명                                                                               | 주요 값                                             |
| :------------ | :--------------------------------------------------------------------------------- | :-------------------------------------------------- |
| `flex-grow`   | 컨테이너에 여유 공간이 있을 때, 아이템이 늘어나는 비율을 설정합니다. (기본값: `0`) | 숫자 (e.g., `1`, `2`)                               |
| `flex-shrink` | 컨테이너 공간이 부족할 때, 아이템이 줄어드는 비율을 설정합니다. (기본값: `1`)      | 숫자 (e.g., `0`, `1`)                               |
| `flex-basis`  | 아이템의 기본 크기를 설정합니다. (기본값: `auto`)                                  | `px`, `%`, `rem` 등 단위 또는 `auto`                |
| `flex`        | `flex-grow`, `flex-shrink`, `flex-basis`를 한 번에 쓰는 단축 속성입니다.           | `0 1 auto` (기본값), `1 1 0`, `none` 등             |
| `order`       | 아이템의 시각적 순서를 변경합니다. 숫자가 작을수록 앞에 옵니다. (기본값: `0`)      | 정수 (e.g., `-1`, `1`, `99`)                        |
| `align-self`  | 개별 아이템의 교차축 정렬을 `align-items`보다 우선하여 설정합니다.                 | `auto` (기본), `flex-start`, `center`, `stretch` 등 |

### `flex-grow` 예시

```html
<style>
  .container {
    display: flex;
    width: 100%;
  }
  .item {
    padding: 1rem;
    border: 1px solid #ccc;
  }
  .item-1 {
    flex-grow: 1;
    background: lightblue;
  }
  .item-2 {
    flex-grow: 2;
    background: lightgreen;
  }
</style>

<div class="container">
  <div class="item item-1">Item 1 (grow: 1)</div>
  <div class="item item-2">Item 2 (grow: 2)</div>
</div>
<!-- Item 2가 Item 1보다 2배 더 많은 여유 공간을 차지합니다. -->
```

---

## 4. 심화 학습 자료

Flexbox는 직접 속성을 바꿔보며 익히는 것이 가장 효과적입니다. 아래 자료들을 통해 개념을 확실히 다져보세요.

- **[A Guide to Flexbox (CSS-Tricks)](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)**

  - Flexbox의 모든 속성을 시각적인 예제와 함께 설명하는 최고의 가이드입니다. 개발 중 필요할 때마다 참고하기 좋습니다.

- **[Flexbox Froggy](https://flexboxfroggy.com/#ko)**
  - 게임을 통해 Flexbox 속성을 익힐 수 있는 재미있는 학습 도구입니다. 개구리를 올바른 위치로 옮기면서 `justify-content`, `align-items` 등의 속성을 자연스럽게 익힐 수 있습니다.

Flexbox는 현대적인 웹 레이아웃의 필수 기술입니다. 다음 문서에서는 2차원 레이아웃을 위한 Grid 시스템에 대해 알아보겠습니다.
