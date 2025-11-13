# 03 쿠키 보안 및 CSRF

#쿠키 #Cookie #CSRF

## 🍪 Spring Boot + Cookie 실습

### 1. 서버 (Spring Boot 3.x)

```java
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "http://127.0.0.1:5500", allowCredentials = "true")
public class CookieController {

    // 단순 쿠키
    @GetMapping("/set-cookie")
    public ResponseEntity<String> setCookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("simpleCookie", "basic-value");
        cookie.setPath("/");
        cookie.setHttpOnly(false); // JS 접근 가능 (보안 취약)
        response.addCookie(cookie);
        return ResponseEntity.ok("일반 쿠키 설정 완료");
    }

    // 보안 쿠키
    @GetMapping("/set-secure-cookie")
    public ResponseEntity<String> setSecureCookie(HttpServletResponse response) {
        ResponseCookie cookie = ResponseCookie.from("secureCookie", "safe-value")
                .httpOnly(true)      // JS 접근 차단 → XSS 방어
                .secure(true)        // HTTPS 필요 (단, localhost/127.0.0.1은 최신 브라우저에서 허용)
                .sameSite("None")    // 크로스 도메인 요청 허용
                .path("/")
                .maxAge(60 * 60)     // 1시간
                .build();

        response.addHeader("Set-Cookie", cookie.toString());
        return ResponseEntity.ok("보안 쿠키 설정 완료");
    }

    @GetMapping("/get-cookie")
    public ResponseEntity<String> getCookie(@CookieValue(value = "secureCookie", required = false) String value) {
        return ResponseEntity.ok("쿠키 값: " + value);
    }
}
```

### 2. 클라이언트 (Vanilla JS, `index.html`)

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>Cookie 테스트</title>
  </head>
  <body>
    <h1>Cookie 테스트</h1>
    <button onclick="setSimpleCookie()">일반 쿠키 설정</button>
    <button onclick="setSecureCookie()">보안 쿠키 설정</button>
    <button onclick="getCookie()">쿠키 확인</button>
    <pre id="result"></pre>

    <script>
      const serverUrl = "http://localhost:8080/api";

      function setSimpleCookie() {
        fetch(`${serverUrl}/set-cookie`, { credentials: "include" })
          .then((res) => res.text())
          .then((txt) => (document.querySelector("#result").textContent = txt));
      }

      function setSecureCookie() {
        fetch(`${serverUrl}/set-secure-cookie`, { credentials: "include" })
          .then((res) => res.text())
          .then((txt) => (document.querySelector("#result").textContent = txt));
      }

      function getCookie() {
        fetch(`${serverUrl}/get-cookie`, { credentials: "include" })
          .then((res) => res.text())
          .then((txt) => (document.querySelector("#result").textContent = txt));
      }
    </script>
  </body>
</html>
```

## 3. Cookie 보안 특성 정리

- **HttpOnly**: JS에서 쿠키 접근 불가. XSS 공격에 안전.
- **Secure**: HTTPS 연결에서만 전송. 단, 최신 브라우저는 `localhost`와 `127.0.0.1`에 한해 예외적으로 허용.
- **SameSite**:
  - `Lax`(기본): 크로스 사이트 요청 중 `일반 네비게이션(GET 링크 클릭 등)`에서는 쿠키 포함, 하지만 자동 전송되는 POST, iframe, fetch, 이미지 요청에는 쿠키가 안 붙음. CSRF에 기본 방어 효과.
  - `Strict`: 완전히 같은 사이트에서만 쿠키 전송. 로그인 유지가 불편해짐.
  - `None`: 크로스 도메인 요청 허용 (반드시 `Secure` 필요).
- **Path / Domain**: 쿠키 적용 범위 지정. 보통 `"/"`로 전체 경로 허용.
- **MaxAge / Expires**: 세션 쿠키(브라우저 닫을 때 삭제) vs 영속 쿠키(만료 시간 지정).

## 4. CSRF (Cross-Site Request Forgery)

- **개념**: 사용자가 로그인된 상태에서 공격자가 의도치 않은 요청을 특정 사이트에 보내도록 속이는 공격 기법. 예를 들어, 사용자가 은행에 로그인 중일 때 공격자가 조작된 폼을 자동 제출해 이체 요청을 발생시키는 경우.
- **쿠키와의 관계**: 브라우저는 기본적으로 같은 도메인의 쿠키를 자동 전송하기 때문에, 공격자가 만든 요청에도 인증 쿠키가 포함될 수 있음. 이 때문에 `SameSite=Lax` 또는 `Strict`는 CSRF에 대한 1차 방어막 역할을 한다.
- **대응 방법**:
  - `SameSite` 속성 활용 (`Lax` 또는 `Strict` 설정).
  - 서버에서 CSRF 토큰을 발급해 요청 시 검증.
  - 중요한 요청(POST/PUT/DELETE)에 대해서는 Referer/Origin 검증.

## 5. 로컬 개발 & 배포 차이

| 환경                         | Secure 쿠키 허용 여부     |
| ---------------------------- | ------------------------- |
| http://localhost             | ✅ 최신 브라우저에서 허용 |
| http://127.0.0.1             | ✅ 최신 브라우저에서 허용 |
| http://192.168.x.x (사설 IP) | ❌ HTTPS 없으면 불가      |
| 배포 환경 (실도메인)         | ✅ 단, 반드시 HTTPS 필요  |

👉 **결론**: 개발 단계는 localhost/127.0.0.1에서 Secure 쿠키 가능, 배포는 무조건 HTTPS + Secure + SameSite(None) + CSRF 방어 적용.
