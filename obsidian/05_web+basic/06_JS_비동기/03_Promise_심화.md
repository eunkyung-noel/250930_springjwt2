# 03. 프로미스 심화: 정적 메서드

#프로미스 #promise #promiseall #promiserace #promisefinally

프로미스는 `.then()`, `.catch()` 외에도 여러 비동기 작업을 효율적으로 관리할 수 있는 유용한 정적 메서드(static methods)를 제공합니다. 대표적으로 `Promise.all()`, `Promise.race()`, `Promise.finally()`가 있습니다.

---

## 1. `Promise.all(iterable)`

- **정의**: 여러 개의 프로미스를 **동시에 실행**하고, **모든 프로미스가 `Fulfilled` 상태가 될 때까지** 기다립니다.
- **동작**:
  - 인자로 받은 모든 프로미스가 성공하면, 각 프로미스의 결과값을 모은 **배열**을 결과로 하는 새로운 프로미스를 반환합니다. (결과 배열의 순서는 인자로 전달된 프로미스의 순서와 동일합니다.)
  - 프로미스 중 **하나라도 `Rejected` 상태가 되면**, 그 즉시 전체 프로미스는 `Rejected` 상태가 되며, 가장 먼저 실패한 프로미스의 에러를 반환합니다.
- **용도**: 서로 의존성이 없는 여러 개의 비동기 작업을 한 번에 처리하고 싶을 때 유용합니다. (예: 여러 API를 동시에 호출하여 모든 데이터가 준비되었을 때 화면을 렌더링하는 경우)

**코드 예시:**

```javascript
// 딜레이를 주는 프로미스 생성 함수
function delay(ms, value) {
  return new Promise((resolve) =>
    setTimeout(() => {
      console.log(`${ms}ms 후 '${value}' 반환`);
      resolve(value);
    }, ms)
  );
}

const promise1 = delay(1000, "첫 번째");
const promise2 = delay(2000, "두 번째");
const promise3 = delay(1500, "세 번째");

// 모든 프로미스가 성공하는 경우
Promise.all([promise1, promise2, promise3])
  .then((results) => {
    // 모든 프로미스가 완료된 후 실행
    // results는 ['첫 번째', '두 번째', '세 번째']가 됩니다.
    console.log("모두 성공! 결과:", results);
  })
  .catch((error) => {
    console.error("실패 발생:", error);
  });

// 하나가 실패하는 경우
const failingPromise = new Promise((_, reject) =>
  setTimeout(() => reject(new Error("실패!")), 1200)
);

Promise.all([delay(1000, "성공1"), failingPromise, delay(1500, "성공2")])
  .then((results) => {
    // 이 부분은 실행되지 않음
    console.log("모두 성공! 결과:", results);
  })
  .catch((error) => {
    // failingPromise가 reject되면 즉시 이 부분이 실행됨
    console.error("실패 발생:", error.message);
  });
```

---

## 2. `Promise.race(iterable)`

- **정의**: 여러 개의 프로미스 중 **가장 먼저 `Settled`(Fulfilled 또는 Rejected) 상태가 되는 프로미스**를 기다립니다.
- **동작**:
  - 가장 먼저 성공한 프로미스가 있다면, 그 프로미스의 결과값을 그대로 반환하는 `Fulfilled` 프로미스가 됩니다.
  - 가장 먼저 실패한 프로미스가 있다면, 그 프로미스의 에러를 그대로 반환하는 `Rejected` 프로미스가 됩니다.
- **용도**: 여러 작업 중 가장 빠른 결과를 사용하고 싶을 때, 또는 특정 시간 내에 작업이 완료되지 않으면 타임아웃 처리를 하고 싶을 때 유용합니다.

**코드 예시:**

```javascript
function delay(ms, value) {
  return new Promise((resolve) =>
    setTimeout(() => {
      console.log(`${ms}ms 후 '${value}' 반환`);
      resolve(value);
    }, ms)
  );
}

// 가장 빠른 프로미스가 이김
Promise.race([delay(2000, "느림"), delay(1000, "빠름")])
  .then((winner) => {
    // 1초 후 '빠름' 프로미스가 resolve되므로, 이 부분이 실행됨
    console.log("승자:", winner); // "승자: 빠름"
  })
  .catch((error) => {
    console.error("패자:", error);
  });

// 타임아웃 처리 예시
function networkRequest() {
  return new Promise((resolve) =>
    setTimeout(() => resolve("데이터 수신 완료"), 3000)
  );
}

function timeout(ms) {
  return new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`${ms}ms 타임아웃!`)), ms)
  );
}

Promise.race([networkRequest(), timeout(2000)])
  .then((data) => {
    console.log(data);
  })
  .catch((error) => {
    // networkRequest(3초)보다 timeout(2초)이 빠르므로, 이 부분이 실행됨
    console.error(error.message); // "2000ms 타임아웃!"
  });
```

---

## 3. `.finally(onFinally)`

- **정의**: 프로미스가 `Fulfilled` 되든 `Rejected` 되든, **성공/실패 여부와 관계없이 항상 마지막에 실행**되는 콜백을 등록합니다.
- **동작**:
  - `.finally()`는 인자를 받지 않으며, 이전 프로미스의 결과나 에러를 건드리지 않고 그대로 다음 체인으로 전달합니다.
- **용도**: 로딩 스피너 숨기기, 데이터베이스 연결 종료 등 결과와 상관없이 항상 수행해야 하는 마무리 작업을 처리할 때 유용합니다.

**코드 예시:**

```javascript
function processData(shouldSucceed) {
  return new Promise((resolve, reject) => {
    console.log("데이터 처리 중..."); // 로딩 스피너 표시
    setTimeout(() => {
      if (shouldSucceed) {
        resolve("처리 성공");
      } else {
        reject("처리 실패");
      }
    }, 1500);
  });
}

// 성공하는 경우
processData(true)
  .then((result) => {
    console.log("결과:", result);
  })
  .catch((error) => {
    console.error("에러:", error);
  })
  .finally(() => {
    // 성공하든 실패하든 항상 실행됨
    console.log("마무리 작업. (로딩 스피너 숨기기)");
  });

// 실패하는 경우
processData(false)
  .then((result) => {
    console.log("결과:", result);
  })
  .catch((error) => {
    console.error("에러:", error);
  })
  .finally(() => {
    // 성공하든 실패하든 항상 실행됨
    console.log("마무리 작업. (로딩 스피너 숨기기)");
  });
```
