# 04. async / await

#async #await #어싱크 #어웨이트

ES2017(ES8)에서 도입된 `async/await`는 프로미스(Promise)를 기반으로, 비동기 코드를 마치 동기 코드처럼 보이게 만들어주는 "Syntactic Sugar"(문법적 설탕)입니다. 복잡한 프로미스 체이닝을 더 직관적이고 가독성 높은 코드로 작성할 수 있게 해줍니다.

---

## 1. 기본 문법

### 가. `async`

- 함수 선언 앞에 `async` 키워드를 붙이면, 해당 함수는 항상 **프로미스를 반환**하는 비동기 함수가 됩니다.
- 만약 함수가 명시적으로 프로미스를 반환하지 않으면, `async` 함수는 그 반환값을 `resolve`하는 프로미스를 자동으로 만들어 반환합니다.

```javascript
// 1. 일반 값을 반환하는 경우
async function getName() {
  return "Alice"; // 이 함수는 "Alice"를 resolve하는 프로미스를 반환합니다.
}

getName().then((name) => console.log(name)); // 출력: Alice

// 2. 프로미스를 반환하는 경우
async function fetchUser() {
  return new Promise((resolve) => {
    setTimeout(() => resolve({ id: 1, name: "Bob" }), 1000);
  });
}

fetchUser().then((user) => console.log(user)); // 출력: { id: 1, name: 'Bob' }
```

### 나. `await`

- `await` 키워드는 **`async` 함수 내부에서만** 사용할 수 있습니다.
- 프로미스 앞에 `await`를 붙이면, 해당 프로미스가 `Settled`(완료 또는 거부) 상태가 될 때까지 함수의 실행을 **일시 중지**하고 기다립니다.
- 프로미스가 `Fulfilled`(성공)되면, `await`는 프로미스의 결과값(resolve된 값)을 반환합니다.
- 프로미스가 `Rejected`(실패)되면, `await`는 에러를 던집니다(throw). 이 에러는 `try...catch` 문으로 잡을 수 있습니다.

---

## 2. `async/await`를 사용한 비동기 처리

프로미스 체이닝으로 작성했던 코드를 `async/await`로 바꿔보겠습니다.

**개선 전 (프로미스 체이닝):**

```javascript
// function addOnePromise(value) { ... }

// addOnePromise(0)
//   .then(result1 => addOnePromise(result1))
//   .then(result2 => addOnePromise(result2))
//   .then(result3 => addOnePromise(result3))
//   .then(result4 => console.log("최종 결과:", result4));
```

**개선 후 (`async/await`):**

```javascript
// 1초 뒤에 값을 1 더하고 resolve하는 프로미스를 반환하는 함수
function addOnePromise(value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      const result = value + 1;
      console.log(`현재 값: ${value}, 1 더한 결과: ${result}`);
      resolve(result);
    }, 1000);
  });
}

// async 함수로 전체 로직을 감싼다
async function runTasks() {
  console.log("작업 시작");

  // await를 사용해 프로미스가 끝날 때까지 기다린다.
  const result1 = await addOnePromise(0);
  const result2 = await addOnePromise(result1);
  const result3 = await addOnePromise(result2);
  const result4 = await addOnePromise(result3);

  console.log("최종 결과:", result4);
}

runTasks();
```

코드가 `.then()`의 중첩 없이 위에서 아래로 순차적으로 실행되는 것처럼 보여 훨씬 이해하기 쉽습니다.

---

## 3. 에러 핸들링: `try...catch`

`async/await` 구문에서 에러를 처리하는 가장 일반적인 방법은 동기 코드에서와 마찬가지로 `try...catch` 문을 사용하는 것입니다.

- `try` 블록 안에서 `await`를 사용한 코드를 실행합니다.
- 만약 프로미스가 `Rejected`되면, `await`는 에러를 던지고 실행 흐름은 즉시 `catch` 블록으로 이동합니다.

**코드 예시:**

```javascript
function createPromise(taskSuccess) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (taskSuccess) {
        resolve("작업 성공!");
      } else {
        reject(new Error("작업 실패!"));
      }
    }, 1000);
  });
}

async function process() {
  try {
    // try 블록에서 비동기 작업을 실행
    const result1 = await createPromise(true);
    console.log("1단계:", result1);

    const result2 = await createPromise(false); // 여기서 에러 발생!
    console.log("2단계:", result2); // 이 코드는 실행되지 않음
  } catch (error) {
    // 에러가 발생하면 catch 블록이 실행됨
    console.error("에러 발생:", error.message);
  } finally {
    // 성공/실패 여부와 관계없이 항상 실행
    console.log("프로세스 종료");
  }
}

process();
```

`try...catch...finally` 구문을 사용하면 동기적인 코드의 에러 처리 방식과 매우 유사하여, 비동기 코드의 에러 흐름을 일관되고 명확하게 관리할 수 있습니다.
