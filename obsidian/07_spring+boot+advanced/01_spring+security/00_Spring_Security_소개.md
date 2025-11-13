# Spring Security 소개

## 1. Spring Security란?

Spring Security는 Spring 기반 애플리케이션의 **인증(Authentication)**과 **인가(Authorization)**를 담당하는 강력하고 유연한 프레임워크입니다. 단순히 말해, "누가 우리 시스템에 접근하려 하는가?"와 "그들이 무엇을 할 수 있는가?"라는 두 가지 핵심 질문을 처리합니다.

- **선언적 보안**: 개발자가 복잡한 보안 로직을 직접 구현하는 대신, 설정을 통해 보안 규칙을 선언적으로 적용할 수 있습니다.
- **서블릿 필터 기반**: Spring Security는 서블릿 필터(Servlet Filter) 체인을 기반으로 동작합니다. 클라이언트의 요청이 애플리케이션의 핵심 로직(Controller)에 도달하기 전에 여러 보안 필터를 거치면서 인증 및 인가 검사를 수행합니다.
- **자동 설정**: `spring-boot-starter-security` 의존성을 추가하는 것만으로도 기본적인 보안 기능(예: 모든 요청에 대한 인증 요구, 기본 로그인 페이지 제공)이 자동으로 활성화됩니다.

---

## 2. 핵심 개념: 인증(Authentication) vs 인가(Authorization)

보안을 이해하는 데 가장 기본이 되는 두 가지 개념입니다.

| 구분                      | 핵심 질문                            | 설명                                                                                     | 예시                                                                                                                                                                        |
| ------------------------- | ------------------------------------ | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **인증 (Authentication)** | "당신은 누구인가?"                   | 사용자의 신원을 확인하는 과정입니다. 시스템에 등록된 사용자인지를 증명하는 절차입니다.   | - 아이디와 비밀번호를 사용한 로그인<br>- 소셜 미디어 계정(Google, GitHub)을 통한 로그인<br>- 지문, 얼굴 인식 등 생체 정보를 이용한 로그인                                   |
| **인가 (Authorization)**  | "당신이 이 작업을 할 권한이 있는가?" | 인증된 사용자가 특정 리소스나 기능에 접근할 수 있는 권한이 있는지를 확인하는 과정입니다. | - `ADMIN` 역할을 가진 사용자만 관리자 페이지에 접근 허용<br>- 로그인한 사용자만 '내 정보' 페이지 조회 가능<br>- 글 작성자 본인만 해당 글을 수정하거나 삭제할 수 있도록 제한 |

**비유**:

- **인증**: 콘서트장에 들어가기 위해 신분증과 티켓을 보여주는 행위.
- **인가**: VIP 티켓을 가진 사람만 무대 앞 VIP 구역에 들어갈 수 있도록 허용하는 것.

---

## 3. Spring Security 아키텍처: `SecurityFilterChain`

Spring Security 3.x 버전부터 보안 설정의 핵심은 `SecurityFilterChain` 빈(Bean)을 설정하는 것입니다.

- **`SecurityFilterChain`**: 요청이 들어왔을 때 거쳐야 할 보안 필터들의 체인(연쇄)입니다. `CsrfFilter`, `UsernamePasswordAuthenticationFilter`, `AuthorizationFilter` 등 다양한 기본 필터들이 정해진 순서에 따라 요청을 처리합니다.
- **`HttpSecurity`**: `SecurityFilterChain`을 구성하기 위한 DSL(Domain Specific Language)을 제공하는 객체입니다. 개발자는 `HttpSecurity` 객체를 사용하여 람다(lambda) 스타일로 보안 규칙을 연쇄적으로 설정할 수 있습니다.
- **중앙 설정 지점**: `HttpSecurity`를 통해 다음과 같은 주요 보안 규칙을 한 곳에서 설정합니다.
  - URL 경로별 접근 권한 (예: `/admin/**`은 `ADMIN` 역할만)
  - 폼 로그인(Form Login), OAuth 2.0 로그인 등 다양한 인증 방식 설정
  - 로그아웃 처리 방식
  - CSRF(Cross-Site Request Forgery) 방어 기능 활성화/비활성화
  - 세션 관리 정책

**설정 예시 (`SecurityConfig.java`)**:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                // 특정 경로는 누구나 접근 허용
                .requestMatchers("/", "/login", "/register").permitAll()
                // 특정 경로는 'USER' 역할을 가진 사용자만 접근 허용
                .requestMatchers("/mypage").hasRole("USER")
                // 나머지 모든 요청은 인증된 사용자에게만 허용
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                // 커스텀 로그인 페이지 경로 설정
                .loginPage("/login")
                // 로그인 성공 시 이동할 기본 URL
                .defaultSuccessUrl("/", true)
            );
        return http.build();
    }
}
```

이처럼 Spring Security는 복잡한 보안 요구사항을 체계적이고 선언적인 방식으로 관리할 수 있게 해주는 필수 프레임워크입니다.
