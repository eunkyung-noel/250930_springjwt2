# Exercise: JWT 인증 시스템 구축

이 실습을 통해 JWT의 기본 개념부터 실제 프로덕션 수준의 인증 시스템을 구축하는 전체 과정을 단계별로 경험합니다.

## 📝 Exercise 01: JWT 기본 개념 및 구조 이해

### 목표

- JWT의 세 가지 구성 요소(Header, Payload, Signature)를 설명할 수 있다.
- 세션 기반 인증과 토큰 기반 인증의 차이점을 비교하고, 토큰 기반 인증의 장단점을 설명할 수 있다.
- JWT Payload에 포함되는 Registered Claims(예: `iss`, `sub`, `exp`)의 의미를 이해한다.

### 문제

1.  JWT의 구조(Header, Payload, Signature)에 대해 각각의 역할과 포함되는 정보를 설명하세요.
2.  `Stateless`의 의미를 설명하고, 이것이 서버 확장성에 어떤 이점을 주는지 설명하세요.
3.  세션 기반 인증과 비교하여 토큰 기반 인증의 장점 2가지와 단점 2가지를 설명하세요.
4.  JWT의 Payload에 민감한 정보(예: 주민등록번호, 비밀번호)를 담으면 안 되는 이유를 설명하세요.

### 평가 기준

- 각 구성 요소의 역할과 주요 정보를 정확히 설명했는가?
- `Stateless` 개념과 서버 확장성의 연관 관계를 명확히 이해하고 있는가?
- 두 인증 방식의 핵심적인 차이점과 장단점을 실용적인 관점에서 비교했는가?
- JWT의 보안적 한계를 인지하고 있는가?

---

## 🛠️ Exercise 02: Spring Security와 JWT 기본 구현

### 목표

- `spring-boot-starter-oauth2-resource-server`를 사용하여 기본적인 JWT 인증 시스템을 구축할 수 있다.
- `JwtEncoder`와 `JwtDecoder`를 설정하고 사용하여 JWT를 생성하고 검증할 수 있다.
- `SecurityFilterChain`을 구성하여 특정 경로는 허용하고 나머지는 인증을 요구하도록 설정할 수 있다.

### 문제

1.  Spring Boot 3.x 프로젝트를 생성하고, `web`, `security`, `oauth2-resource-server`, `oauth2-jose` 의존성을 추가하세요.
2.  `application.yml`에 HS256 알고리즘을 위한 32바이트 이상의 `jwt.secret-key`를 설정하세요.
3.  `SecurityConfig` 클래스를 생성하고 다음을 구현하세요.
    - `SessionCreationPolicy.STATELESS`로 세션 정책을 설정하세요.
    - `/auth/login` 경로는 모두에게 허용하고, 나머지 모든 요청은 인증을 요구하도록 설정하세요.
    - `JwtEncoder`와 `JwtDecoder` Bean을 `jwt.secret-key`를 사용하여 등록하세요.
    - `oauth2ResourceServer`를 사용하여 JWT 인증을 활성화하세요.
4.  `AuthController`를 생성하여 `/auth/login` 엔드포인트를 구현하세요.
    - 요청 본문으로 받은 ID/PW를 기반으로 사용자를 인증하고(여기서는 In-memory 사용자 사용), 성공 시 `JwtEncoder`를 사용하여 Access Token을 발급하는 코드를 작성하세요.
5.  Postman을 사용하여 `/auth/login`을 호출하여 토큰을 발급받고, 해당 토큰을 `Authorization: Bearer <token>` 헤더에 담아 보호된 API(예: `/api/me`)를 호출하여 `200 OK` 응답을 확인하세요.

### 평가 기준

- 필요한 의존성을 정확히 추가하고 프로젝트를 설정했는가?
- `SecurityConfig`에서 JWT 인증을 위한 핵심 설정을 올바르게 구성했는가?
- `JwtEncoder`를 사용하여 유효한 JWT를 생성하고, `JwtDecoder`를 통해 검증 플로우를 구성했는가?
- API 테스트 도구를 통해 전체 인증 흐름을 성공적으로 시연할 수 있는가?

---

## ⚙️ Exercise 03: 커스텀 JWT 필터 구현

### 목표

- `UsernamePasswordAuthenticationFilter`를 커스터마이징하여 로그인 처리 및 JWT 발급 필터를 구현할 수 있다.
- `OncePerRequestFilter`를 사용하여 들어오는 모든 요청의 JWT를 검증하는 인가 필터를 구현할 수 있다.
- `SecurityFilterChain`에 직접 만든 커스텀 필터들을 올바른 순서로 등록할 수 있다.

### 문제

1.  `JwtAuthenticationFilter`를 `UsernamePasswordAuthenticationFilter`를 상속받아 생성하세요.
    - `attemptAuthentication`: 사용자 인증을 시도합니다.
    - `successfulAuthentication`: 인증 성공 시 JWT를 생성하고, **HttpOnly 쿠키**에 담아 응답에 추가합니다.
2.  `JwtAuthorizationFilter`를 `OncePerRequestFilter`를 상속받아 생성하세요.
    - `doFilterInternal`: 요청의 쿠키에서 JWT를 읽어와 유효성을 검증하고, 유효한 경우 `SecurityContextHolder`에 인증 정보를 설정합니다.
3.  `SecurityConfig`를 수정하여 기존 `oauth2ResourceServer` 설정을 제거하고, 위에서 만든 `JwtAuthenticationFilter`와 `JwtAuthorizationFilter`를 필터 체인에 등록하세요.
    - `JwtAuthenticationFilter`는 `UsernamePasswordAuthenticationFilter` 위치에 등록합니다.
    - `JwtAuthorizationFilter`는 `JwtAuthenticationFilter` 앞에 등록합니다.
4.  로그아웃 기능을 구현하세요. `/logout` 요청 시 `accessToken` 쿠키를 삭제하고 로그인 페이지로 리다이렉트하는 핸들러를 `SecurityConfig`에 추가하세요.
5.  Thymeleaf 또는 간단한 HTML 페이지를 만들어 로그인/로그아웃 흐름을 브라우저에서 테스트하고, 개발자 도구에서 쿠키가 정상적으로 생성되고 삭제되는지 확인하세요.

### 평가 기준

- 두 가지 핵심 JWT 필터의 역할을 이해하고 각각 올바르게 구현했는가?
- `SecurityFilterChain`에 커스텀 필터를 정확한 순서와 위치에 등록했는가?
- HttpOnly 쿠키를 사용하여 안전하게 토큰을 전달하고 관리하는 방법을 구현했는가?
- 브라우저 환경에서 전체 로그인-API요청-로그아웃 흐름을 완벽하게 시연할 수 있는가?

---

## 🌐 Exercise 04: CORS 및 프론트엔드 연동

### 목표

- CORS의 개념과 Preflight 요청의 동작 원리를 이해한다.
- Spring Security 환경에서 `CorsConfigurationSource`를 사용하여 CORS 정책을 설정할 수 있다.
- Vanilla JavaScript 또는 React와 같은 프론트엔드 클라이언트에서 JWT 인증 서버와 통신할 수 있다.

### 문제

1.  별도의 프론트엔드 프로젝트(또는 `Live Server`를 이용한 `index.html`)를 `http://127.0.0.1:5500`에서 실행한다고 가정하고, Spring Boot 애플리케이션(`localhost:8080`)에 CORS 설정을 추가하세요.
    - `SecurityConfig`에 `CorsConfigurationSource` Bean을 등록하여 `http://127.0.0.1:5500` 출처의 요청을 허용하도록 설정하세요.
    - 허용할 메서드(`GET`, `POST` 등)와 헤더(`Authorization` 등)를 명시적으로 설정하세요.
2.  프론트엔드에서 `localStorage`를 사용하여 JWT를 관리하는 로그인/API 호출/로그아웃 기능을 JavaScript로 구현하세요.
    - 로그인 성공 시 서버로부터 받은 토큰을 `localStorage`에 저장합니다.
    - 보호된 API 호출 시 `Authorization: Bearer <token>` 헤더에 토큰을 담아 전송합니다.
    - 로그아웃 시 `localStorage`에서 토큰을 삭제합니다.
3.  (심화) `HttpOnly` 쿠키 방식과 연동하는 프론트엔드 코드를 작성해 보세요.
    - 로그인/로그아웃 요청만 보내고, 토큰 관리는 브라우저에 맡깁니다.
    - API 요청 시 `credentials: 'include'` 옵션을 사용하여 쿠키가 자동으로 전송되도록 설정해야 합니다.

### 평가 기준

- CORS 정책을 올바르게 설정하여 다른 출처의 프론트엔드 요청을 성공적으로 처리할 수 있는가?
- `localStorage` 방식과 `HttpOnly` 쿠키 방식의 프론트엔드 구현 차이점을 이해하고 코드로 표현할 수 있는가?
- 브라우저 개발자 도구의 네트워크 탭을 통해 CORS Preflight 요청과 본 요청의 헤더를 분석하고 설명할 수 있는가?

---

## 🚀 Exercise 05: OAuth2 소셜 로그인과 JWT 통합

### 목표

- Spring Security의 OAuth2 클라이언트 기능을 사용하여 소셜 로그인(Google, GitHub 등)을 연동할 수 있다.
- OAuth2 인증 성공 후, 시스템의 자체 JWT를 발급하여 기존 인증 체계와 통합할 수 있다.
- `CustomOAuth2UserService`를 구현하여 Provider로부터 받은 사용자 정보를 DB에 저장하거나 업데이트할 수 있다.

### 문제

1.  `spring-boot-starter-oauth2-client` 의존성을 추가하고, Google 또는 GitHub의 OAuth2 클라이언트 ID와 시크릿을 `application.yml`에 설정하세요.
2.  `CustomOAuth2UserService`를 구현하여 소셜 로그인 사용자의 정보를 처리하는 로직을 작성하세요.
    - 처음 로그인하는 사용자는 DB에 새로 저장합니다.
    - 기존 사용자는 정보를 업데이트할 수 있습니다.
    - Provider(Google, GitHub)에 따라 상이한 사용자 정보(`attributes`)를 파싱하여 일관된 `User` 객체로 매핑하세요.
3.  `OAuth2AuthenticationSuccessHandler`를 구현하세요.
    - OAuth2 인증이 성공적으로 완료되면, 인증된 사용자 정보(`Authentication` 객체)를 기반으로 우리 시스템의 JWT를 생성합니다.
    - 생성된 JWT를 `HttpOnly` 쿠키에 담아 응답에 추가하고, 프론트엔드의 특정 페이지(예: `/login-success`)로 리다이렉트합니다.
4.  `SecurityConfig`에 `oauth2Login` 설정을 추가하고, 위에서 만든 `CustomOAuth2UserService`와 `OAuth2AuthenticationSuccessHandler`를 연결하세요.
5.  프론트엔드에 "Google로 로그인" 버튼을 추가하고, 전체 소셜 로그인 흐름을 테스트하세요.
    - 로그인 성공 후, 기존 JWT로 보호되던 API가 정상적으로 호출되는지 확인하세요.

### 평가 기준

- OAuth2 인증 코드 플로우를 이해하고 Spring Security에 올바르게 설정했는가?
- `CustomOAuth2UserService`에서 사용자 정보를 DB와 연동하여 영속적으로 관리할 수 있는가?
- `OAuth2AuthenticationSuccessHandler`를 통해 소셜 로그인과 기존 JWT 인증 시스템을 완벽하게 통합했는가?
- 일반 로그인 사용자와 소셜 로그인 사용자 모두 동일한 방식으로 API 접근이 제어되는 것을 시연할 수 있는가?
