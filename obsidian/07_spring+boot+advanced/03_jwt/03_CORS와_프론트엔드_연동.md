# 03. CORS와 프론트엔드 연동

#CORS #Frontend #JavaScript #React #localStorage #HttpOnly

JWT 기반의 인증 서버를 구축했다면, 이제 프론트엔드 클라이언트와 연동해야 합니다. 이때 가장 먼저 마주치는 문제가 바로 **CORS(Cross-Origin Resource Sharing)**입니다.

## 1. CORS (Cross-Origin Resource Sharing)란?

브라우저는 보안상의 이유로 **동일 출처 정책(Same-Origin Policy)**을 따릅니다. 이 정책은 스크립트가 자신이 로드된 출처(Origin)와 다른 출처의 리소스와 상호작용하는 것을 제한합니다.

- **출처(Origin)**: `Protocol` + `Host` + `Port`
  - `http://localhost:8080`과 `http://127.0.0.1:5500`은 포트와 호스트가 다르므로 다른 출처입니다.

CORS는 서버가 특정 출처의 요청을 허용하도록 `Access-Control-Allow-Origin`과 같은 HTTP 헤더를 응답에 포함시켜 이 문제를 해결하는 메커니즘입니다.

### CORS의 동작 방식 (Preflight Request)

`GET`, `HEAD`, `POST`(특정 조건) 이외의 요청(예: `PUT`, `DELETE`)이나 커스텀 헤더(`Authorization` 등)를 포함하는 요청 시, 브라우저는 본 요청을 보내기 전에 `OPTIONS` 메서드를 사용하여 **Preflight Request**를 먼저 보냅니다.

1.  **Client (Browser)** -> **Server**: `OPTIONS /api/resource`
    - `Origin`: `http://127.0.0.1:5500`
    - `Access-Control-Request-Method`: `PUT`
    - `Access-Control-Request-Headers`: `Authorization`
2.  **Server** -> **Client**: `200 OK`
    - `Access-Control-Allow-Origin`: `http://127.0.0.1:5500`
    - `Access-Control-Allow-Methods`: `GET, POST, PUT, DELETE`
    - `Access-Control-Allow-Headers`: `Authorization, Content-Type`
3.  (Preflight 성공 시) **Client** -> **Server**: `PUT /api/resource` (본 요청)

## 2. Spring Security에서 CORS 설정

`SecurityConfig`에서 `CorsConfigurationSource` Bean을 등록하여 CORS 설정을 중앙에서 관리할 수 있습니다.

```java
// config/SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();

        // 허용할 출처 설정
        config.setAllowedOrigins(List.of("http://127.0.0.1:5500", "http://localhost:3000"));

        // 허용할 HTTP 메서드 설정
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"));

        // 허용할 HTTP 헤더 설정
        config.setAllowedHeaders(List.of("*"));

        // 자격 증명(쿠키 등) 허용 여부
        config.setAllowCredentials(true);

        // Preflight 요청의 캐시 시간 (초)
        config.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config); // 모든 경로에 대해 위 설정 적용
        return source;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.cors(cors -> cors.configurationSource(corsConfigurationSource())); // CORS 설정 적용
        // ... (나머지 설정)
        return http.build();
    }
}
```

## 3. 프론트엔드 연동 예제

### JWT 저장 전략: `localStorage` vs. `HttpOnly` 쿠키

| 구분     | `localStorage`                                                               | `HttpOnly` 쿠키                                                             |
| :------- | :--------------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| **장점** | - 구현이 매우 간단<br>- 모바일 앱 등 다른 환경과 일관된 방식 사용 가능       | - XSS 공격으로부터 토큰 탈취 방지<br>- CSRF는 `SameSite` 속성으로 방어 가능 |
| **단점** | - XSS 공격에 취약 (JS로 토큰 접근 가능)<br>- CSRF 보호를 위한 추가 구현 필요 | - 구현이 상대적으로 복잡<br>- 다른 도메인에서 쿠키 사용 시 설정 까다로움    |
| **구현** | 서버는 JSON으로 토큰 전달, 클라이언트가 JS로 저장                            | 서버가 `Set-Cookie` 헤더로 전달, 브라우저가 자동 관리                       |

### 예제 1: Vanilla JavaScript + `localStorage`

이 방식은 구현이 간단하여 프로토타이핑이나 간단한 애플리케이션에 적합합니다.

#### `index.html`

```html
<!DOCTYPE html>
<html lang="ko">
  <body>
    <h1>JWT 클라이언트 (localStorage)</h1>
    <input id="username" placeholder="Username" value="user" />
    <input
      id="password"
      type="password"
      placeholder="Password"
      value="password"
    />
    <button id="loginBtn">로그인</button>
    <button id="getDataBtn">보호된 데이터 요청</button>
    <button id="logoutBtn">로그아웃</button>
    <pre id="result"></pre>
    <script src="app.js"></script>
  </body>
</html>
```

#### `app.js`

```javascript
const loginBtn = document.getElementById("loginBtn");
const getDataBtn = document.getElementById("getDataBtn");
const logoutBtn = document.getElementById("logoutBtn");
const resultDiv = document.getElementById("result");

const API_BASE_URL = "http://localhost:8080";

// 로그인
loginBtn.addEventListener("click", async () => {
  const username = document.getElementById("username").value;
  const password = document.getElementById("password").value;

  const response = await fetch(`${API_BASE_URL}/auth/login`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ username, password }),
  });

  if (response.ok) {
    const data = await response.json();
    localStorage.setItem("accessToken", data.accessToken);
    resultDiv.textContent =
      "로그인 성공! 토큰이 localStorage에 저장되었습니다.";
  } else {
    resultDiv.textContent = "로그인 실패";
  }
});

// 보호된 데이터 요청
getDataBtn.addEventListener("click", async () => {
  const token = localStorage.getItem("accessToken");
  if (!token) {
    resultDiv.textContent = "토큰이 없습니다. 먼저 로그인하세요.";
    return;
  }

  const response = await fetch(`${API_BASE_URL}/api/me`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (response.ok) {
    const data = await response.json();
    resultDiv.textContent = JSON.stringify(data, null, 2);
  } else {
    resultDiv.textContent = "데이터 요청 실패: " + response.statusText;
  }
});

// 로그아웃
logoutBtn.addEventListener("click", () => {
  localStorage.removeItem("accessToken");
  resultDiv.textContent = "로그아웃 되었습니다.";
});
```

### 예제 2: React + `HttpOnly` 쿠키

이 방식은 서버가 쿠키를 관리하므로 클라이언트 코드가 더 깔끔해지고 보안에 강합니다. 클라이언트는 별도로 토큰을 관리할 필요 없이 그냥 API를 호출하면 됩니다.

#### `AuthManager.js` (React Custom Hook)

```jsx
// hooks/useAuth.js
import { useState, useEffect } from "react";
import axios from "axios";

const API_BASE_URL = "http://localhost:8080";

// axios 인스턴스 생성 (쿠키 전송을 위해 withCredentials 설정)
const apiClient = axios.create({
  baseURL: API_BASE_URL,
  withCredentials: true,
});

export function useAuth() {
  const [user, setUser] = useState(null);

  // 로그인 함수
  const login = async (username, password) => {
    try {
      await apiClient.post("/auth/login", { username, password });
      // 로그인 성공 후 사용자 정보 다시 가져오기
      await fetchUser();
    } catch (error) {
      console.error("Login failed:", error);
      setUser(null);
    }
  };

  // 로그아웃 함수
  const logout = async () => {
    try {
      await apiClient.post("/auth/logout"); // 서버에 로그아웃 요청
      setUser(null);
    } catch (error) {
      console.error("Logout failed:", error);
    }
  };

  // 사용자 정보 가져오는 함수
  const fetchUser = async () => {
    try {
      const response = await apiClient.get("/api/me");
      setUser(response.data);
    } catch (error) {
      // 401 Unauthorized 등 에러 발생 시 사용자 null 처리
      setUser(null);
    }
  };

  // 컴포넌트 마운트 시 사용자 정보 가져오기
  useEffect(() => {
    fetchUser();
  }, []);

  return { user, login, logout, fetchUser };
}
```

#### `App.js` (React Component)

```jsx
import React from "react";
import { useAuth } from "./hooks/useAuth";

function App() {
  const { user, login, logout } = useAuth();

  const handleLogin = () => {
    login("user", "password");
  };

  return (
    <div>
      <h1>JWT 클라이언트 (HttpOnly Cookie)</h1>
      {user ? (
        <div>
          <p>환영합니다, {user.username}님!</p>
          <pre>{JSON.stringify(user, null, 2)}</pre>
          <button onClick={logout}>로그아웃</button>
        </div>
      ) : (
        <div>
          <p>로그인되지 않았습니다.</p>
          <button onClick={handleLogin}>로그인</button>
        </div>
      )}
    </div>
  );
}

export default App;
```

이처럼 프론트엔드에서는 JWT 저장 전략에 따라 구현 방식이 달라집니다. 보안을 중시하는 프로덕션 환경에서는 `HttpOnly` 쿠키와 Refresh Token 패턴을 함께 사용하는 것이 권장됩니다.
