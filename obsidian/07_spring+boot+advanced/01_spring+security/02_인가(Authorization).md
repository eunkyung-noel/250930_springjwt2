# 인가 (Authorization)

인가는 **인증(Authentication)**을 통해 신원이 확인된 사용자가 특정 리소스나 기능에 접근할 수 있는 **권한(Authority)**이 있는지를 확인하는 과정입니다. Spring Security에서는 `SecurityFilterChain` 내의 `authorizeHttpRequests` DSL(Domain Specific Language)을 통해 매우 직관적이고 강력한 인가 정책을 설정할 수 있습니다.

---

## 1. `authorizeHttpRequests` DSL을 이용한 접근 제어

`SecurityConfig`의 `filterChain` 메서드 안에서 `http.authorizeHttpRequests(...)`를 호출하여 요청별 인가 규칙을 설정합니다. 규칙은 위에서부터 순서대로 적용되며, 가장 먼저 매칭되는 규칙이 우선권을 갖습니다. 따라서 **구체적인 경로를 먼저 설정하고 포괄적인 경로는 나중에 설정**하는 것이 중요합니다.

**설정 구조**:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                // 1. 구체적인 경로에 대한 규칙
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/mypage", "/settings").hasRole("USER")
                .requestMatchers("/", "/login", "/register", "/css/**", "/js/**").permitAll()
                // 2. 나머지 모든 요청에 대한 규칙 (가장 마지막에 위치)
                .anyRequest().authenticated()
            );
        // ... formLogin, logout 등 다른 설정 ...
        return http.build();
    }
}
```

---

## 2. 주요 인가 관련 메서드

`authorizeHttpRequests` 람다 표현식 내에서 사용할 수 있는 주요 메서드는 다음과 같습니다.

| 메서드                            | 설명                                                                                                                                                            | 예시                                                                |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `requestMatchers(patterns...)`    | 인가 규칙을 적용할 URL, HTTP 메서드 등의 패턴을 지정합니다. Ant 스타일 패턴(`*`, `**`)을 사용할 수 있습니다.                                                    | `.requestMatchers(HttpMethod.POST, "/api/posts")`                   |
| `.permitAll()`                    | 지정된 패턴에 대해 **모든 사용자**(인증 여부와 관계없이)의 접근을 허용합니다. 주로 홈페이지, 로그인/회원가입 페이지, 정적 리소스(CSS, JS, 이미지)에 사용됩니다. | `.requestMatchers("/", "/login", "/images/**").permitAll()`         |
| `.denyAll()`                      | 지정된 패턴에 대해 **모든 사용자**의 접근을 차단합니다.                                                                                                         | `.requestMatchers("/internal-api/**").denyAll()`                    |
| `.authenticated()`                | **인증된 사용자**만 접근을 허용합니다. 역할(Role)에 관계없이 로그인만 되어 있으면 접근할 수 있습니다.                                                           | `.requestMatchers("/dashboard").authenticated()`                    |
| `.hasRole("USER")`                | 지정된 **역할(Role)**을 가진 사용자만 접근을 허용합니다. `ROLE_` 접두사는 자동으로 추가되므로, 역할 이름만 전달하면 됩니다.                                     | `.requestMatchers("/mypage").hasRole("USER")`                       |
| `.hasAnyRole("ADMIN", "MANAGER")` | 여러 역할 중 **하나라도** 가진 사용자에게 접근을 허용합니다.                                                                                                    | `.requestMatchers("/management/**").hasAnyRole("ADMIN", "MANAGER")` |
| `.hasAuthority("SCOPE_read")`     | `GrantedAuthority`를 직접 비교합니다. 역할(Role)보다 더 세분화된 권한(Authority)을 검사할 때 사용됩니다. OAuth 2.0의 스코프(Scope) 검증 등에 유용합니다.        | `.requestMatchers("/api/resource").hasAuthority("SCOPE_read")`      |
| `.access(expression)`             | SpEL(Spring Expression Language)을 사용하여 복잡한 인가 규칙을 직접 작성할 수 있습니다.                                                                         | `.access("hasRole('ADMIN') and hasIpAddress('192.168.0.0/16')")`    |

---

## 3. 역할(Role)과 권한(Authority)의 관계

- **권한 (Authority)**: 시스템에서 수행할 수 있는 개별적인 행위 (예: `read`, `write`, `delete_post`). `GrantedAuthority` 인터페이스로 표현됩니다.
- **역할 (Role)**: 권한들의 집합. 특정 역할을 가진 사용자는 해당 역할에 부여된 모든 권한을 갖습니다 (예: `ADMIN` 역할은 `read`, `write`, `delete` 권한을 모두 가짐).

Spring Security에서 `hasRole("ADMIN")`은 내부적으로 `hasAuthority("ROLE_ADMIN")`과 동일하게 동작합니다. 즉, 역할은 `ROLE_` 접두사가 붙은 특별한 형태의 권한입니다. `UserDetailsService`에서 `GrantedAuthority`를 생성할 때 `ROLE_` 접두사를 붙이는 이유가 바로 이 때문입니다.

**권한 생성 예시**:

```java
// UserDetails 구현체 내부
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    // "ROLE_USER"라는 권한을 생성하여 반환
    return Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"));
}
```

이처럼 `authorizeHttpRequests`를 통해 선언적으로 인가 규칙을 정의함으로써, 애플리케이션의 보안 요구사항을 명확하고 간결하게 관리할 수 있습니다.
