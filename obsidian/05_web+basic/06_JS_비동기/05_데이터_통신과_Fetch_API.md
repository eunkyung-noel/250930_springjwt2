# 05. 데이터 통신과 Fetch API

#fetch #api #json #http

웹 프론트엔드 개발의 핵심적인 부분은 서버와 데이터를 주고받는 것입니다. 현대 자바스크립트에서는 `Fetch API`를 사용하여 HTTP 요청을 보내고 서버로부터 데이터를 받아오는 작업을 간편하게 처리할 수 있습니다. 이 과정에서 주로 사용되는 데이터 형식은 `JSON`입니다.

---

## 1. HTTP 통신 기본

- **HTTP(HyperText Transfer Protocol)**: 웹에서 클라이언트(브라우저)와 서버가 서로 데이터를 주고받기 위해 사용하는 통신 규약입니다.
- **요청(Request)과 응답(Response)**: 클라이언트가 서버에 특정 정보를 요청하면, 서버는 그 요청을 처리하고 응답을 보냅니다.
- **HTTP 메서드**: 요청의 종류를 나타냅니다.
  - `GET`: 서버로부터 데이터를 **조회**할 때 사용합니다. (예: 게시글 목록 보기)
  - `POST`: 서버에 새로운 데이터를 **생성**할 때 사용합니다. (예: 회원 가입, 글쓰기)
  - `PUT` / `PATCH`: 기존 데이터를 **수정**할 때 사용합니다.
  - `DELETE`: 기존 데이터를 **삭제**할 때 사용합니다.

---

## 2. JSON (JavaScript Object Notation)

- **정의**: 데이터를 교환하기 위해 만들어진, 가볍고 읽기 쉬운 텍스트 기반의 데이터 형식입니다. 자바스크립트 객체 문법에서 파생되었지만, 특정 언어에 종속되지 않아 널리 사용됩니다.
- **특징**:
  - `"키": "값"` 형태의 쌍으로 이루어져 있습니다.
  - 키는 반드시 큰따옴표(`"`)로 묶어야 합니다.
  - 값으로는 문자열, 숫자, 불리언, 배열, 다른 JSON 객체가 올 수 있습니다.

**JSON 예시:**

```json
{
  "id": 1,
  "name": "피카츄",
  "type": ["전기"],
  "isAvailable": true,
  "stats": {
    "hp": 35,
    "attack": 55
  }
}
```

### 자바스크립트 객체와 JSON 변환

- `JSON.stringify(object)`: 자바스크립트 객체를 JSON 형식의 **문자열**로 변환합니다. (서버로 데이터를 보낼 때 사용)
- `JSON.parse(string)`: JSON 형식의 **문자열**을 자바스크립트 객체로 변환합니다. (서버로부터 받은 데이터를 사용할 때 사용)

```javascript
const pokemon = {
  id: 25,
  name: "Pikachu",
};

// 객체 -> JSON 문자열
const jsonString = JSON.stringify(pokemon);
console.log(jsonString); // '{"id":25,"name":"Pikachu"}'

// JSON 문자열 -> 객체
const parsedObject = JSON.parse(jsonString);
console.log(parsedObject.name); // 'Pikachu'
```

---

## 3. Fetch API

- **정의**: `XMLHttpRequest`의 단점을 보완하여 나온, HTTP 요청/응답을 처리하기 위한 강력하고 유연한 자바스크립트 인터페이스입니다. 프로미스(Promise)를 기반으로 동작하여 `async/await`와 함께 사용하기 매우 편리합니다.

### 가. 기본 `GET` 요청

`fetch(url)`를 호출하면 해당 URL에 `GET` 요청을 보내고, 그 결과로 **`Response` 객체를 `resolve`하는 프로미스**를 반환합니다.

- `Response` 객체는 실제 데이터가 아니라, HTTP 응답 전체를 나타내는 객체입니다.
- 실제 JSON 데이터를 얻기 위해서는 `response.json()` 메서드를 추가로 호출해야 합니다. 이 메서드 역시 프로미스를 반환합니다.

**코드 예시 (`async/await` 사용):**

```javascript
// PokeAPI를 사용하여 포켓몬 데이터를 가져오는 예시
async function fetchPokemon(pokemonName) {
  const url = `https://pokeapi.co/api/v2/pokemon/${pokemonName}`;

  try {
    // 1. fetch 요청 보내기 (프로미스 반환)
    const response = await fetch(url);

    // response.ok는 HTTP 상태 코드가 200-299 범위에 있는지 확인합니다.
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    // 2. 응답을 JSON으로 파싱하기 (프로미스 반환)
    const data = await response.json();

    // 3. 데이터 사용하기
    console.log(`이름: ${data.name}`);
    console.log(`타입: ${data.types.map((t) => t.type.name).join(", ")}`);
    console.log(`키: ${data.height * 10} cm`);
  } catch (error) {
    console.error("포켓몬 데이터를 가져오는 데 실패했습니다:", error);
  }
}

fetchPokemon("ditto");
```

### 나. `POST` 요청 및 옵션 설정

`POST` 요청을 보내거나 헤더(header) 등을 설정하려면 `fetch` 함수의 두 번째 인자로 옵션 객체를 전달합니다.

- `method`: HTTP 메서드 (`'POST'`, `'PUT'` 등)
- `headers`: 요청 헤더. 서버와 클라이언트가 어떤 형식의 데이터를 주고받을지 명시합니다.
- `body`: 요청에 담아 보낼 데이터. `JSON.stringify()`를 사용하여 문자열로 변환해야 합니다.

**코드 예시:**

```javascript
async function createPost(newPost) {
  const url = "https://jsonplaceholder.typicode.com/posts";

  try {
    const response = await fetch(url, {
      method: "POST", // 메서드 지정
      headers: {
        // 우리가 보내는 데이터가 JSON 형식임을 알림
        "Content-Type": "application/json",
      },
      // newPost 객체를 JSON 문자열로 변환하여 body에 담아 보냄
      body: JSON.stringify(newPost),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const createdPost = await response.json();
    console.log("새로 생성된 포스트:", createdPost);
  } catch (error) {
    console.error("포스트 생성에 실패했습니다:", error);
  }
}

createPost({
  title: "새로운 글",
  body: "이것은 fetch POST 요청 예시입니다.",
  userId: 1,
});
```
