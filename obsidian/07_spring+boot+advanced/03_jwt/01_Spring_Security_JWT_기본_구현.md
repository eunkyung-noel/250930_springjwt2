# 01. Spring Security와 JWT 기본 구현

#SpringSecurity #JWT #OAuth2ResourceServer #JwtEncoder #JwtDecoder

Spring Boot 3.x와 Spring Security 6.x부터는 JWT 구현이 훨씬 간소화되었습니다. `spring-boot-starter-oauth2-resource-server` 의존성을 활용하면 `JwtEncoder`와 `JwtDecoder`를 통해 손쉽게 JWT를 생성하고 검증할 수 있습니다.

## 1. 의존성 추가

`build.gradle`에 다음 의존성을 추가합니다.

```groovy
// build.gradle
dependencies {
    // Spring Web, Spring Security
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'

    // Spring Security OAuth2 Resource Server (JWT 검증)
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'

    // Spring Security OAuth2 JOSE (JWT 생성)
    // Nimbus-JOSE-JWT 라이브러리를 내부적으로 사용
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-jose'

    // 테스트 의존성
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}
```

- `spring-boot-starter-oauth2-resource-server`: JWT 토큰을 검증하고 인증된 리소스 서버를 구성하는 데 필요한 기능을 제공합니다. `JwtDecoder` Bean을 자동으로 구성해줍니다.
- `spring-boot-starter-oauth2-jose`: JWT를 생성(`JwtEncoder`)하는 데 필요한 라이브러리들을 포함합니다.

## 2. `application.yml` 설정

JWT 비밀키와 같은 민감한 정보는 `application.yml`에 설정합니다. 운영 환경에서는 환경 변수나 외부 설정 파일을 사용하는 것이 안전합니다.

```yaml
# src/main/resources/application.yml

jwt:
  # 32바이트 이상의 무작위 문자열을 사용해야 합니다.
  # 예: openssl rand -hex 32
  secret-key: "your-super-strong-secret-key-for-hs256-must-be-at-least-32-bytes"
  issuer: "demo-app"
  access-token-expiration: 3600 # 초 (1시간)
```

> ⚠️ **HS256 알고리즘**을 사용하려면 비밀키는 **반드시 32바이트(256비트) 이상**이어야 합니다.

## 3. `SecurityConfig` 설정

`SecurityConfig`에서 `JwtEncoder`와 `JwtDecoder`를 설정하고, `SecurityFilterChain`을 구성합니다.

```java
// config/SecurityConfig.java
package com.example.jwt.config;

import com.nimbusds.jose.jwk.source.ImmutableSecret;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtEncoder;
import org.springframework.security.web.SecurityFilterChain;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    @Value("${jwt.secret-key}")
    private String secretKey;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable) // CSRF 보호 비활성화
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // 세션 상태 비저장
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/auth/login").permitAll() // 로그인 경로는 인증 없이 허용
                        .anyRequest().authenticated() // 나머지 경로는 인증 필요
                )
                .oauth2ResourceServer(oauth2 -> oauth2.jwt(jwt -> jwt.decoder(jwtDecoder()))) // JWT 기반 리소스 서버 설정
                .build();
    }

    @Bean
    public JwtEncoder jwtEncoder() {
        SecretKey key = new SecretKeySpec(secretKey.getBytes(), "HmacSHA256");
        return new NimbusJwtEncoder(new ImmutableSecret<>(key));
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        SecretKey key = new SecretKeySpec(secretKey.getBytes(), "HmacSHA256");
        return NimbusJwtDecoder.withSecretKey(key).build();
    }
}
```

### 주요 설정 설명

- `csrf(AbstractHttpConfigurer::disable)`: JWT는 상태를 저장하지 않으므로 CSRF 공격에 비교적 안전합니다. 따라서 CSRF 보호 기능을 비활성화합니다.
- `sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))`: 세션을 사용하지 않고, 각 요청을 독립적으로 처리하도록 설정합니다.
- `authorizeHttpRequests(...)`: URL 경로별 접근 권한을 설정합니다.
- `oauth2ResourceServer(oauth2 -> oauth2.jwt(...))`: Spring Security가 들어오는 요청의 `Authorization: Bearer <token>` 헤더를 확인하고, `jwtDecoder()`로 토큰을 검증하도록 설정합니다. 검증이 성공하면 `Authentication` 객체를 생성하여 `SecurityContext`에 저장합니다.
- `jwtEncoder()`: `application.yml`의 비밀키를 사용하여 JWT를 생성하는 `JwtEncoder`를 Bean으로 등록합니다.
- `jwtDecoder()`: 동일한 비밀키를 사용하여 JWT를 검증하는 `JwtDecoder`를 Bean으로 등록합니다.

## 4. JWT 생성 및 로그인 API 구현

`JwtTokenService`를 만들어 토큰 생성 로직을 분리하고, `AuthController`에서 로그인 API를 구현합니다.

### `JwtTokenService.java`

```java
// service/JwtTokenService.java
package com.example.jwt.service;

import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.oauth2.jwt.JwtClaimsSet;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.JwtEncoderParameters;
import org.springframework.stereotype.Service;

import java.time.Instant;

@Service
@RequiredArgsConstructor
public class JwtTokenService {

    private final JwtEncoder jwtEncoder;

    @Value("${jwt.issuer}")
    private String issuer;

    @Value("${jwt.access-token-expiration}")
    private long accessTokenExpiration;

    public String createAccessToken(String username) {
        Instant now = Instant.now();

        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer(issuer)
                .subject(username)
                .issuedAt(now)
                .expiresAt(now.plusSeconds(accessTokenExpiration))
                // .claim("roles", "USER") // 필요시 커스텀 클레임 추가
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
}
```

### `AuthController.java`

여기서는 간단하게 인메모리 사용자 정보를 사용합니다. 실제 애플리케이션에서는 `UserDetailsService`와 `PasswordEncoder`를 구현하여 DB와 연동해야 합니다.

```java
// controller/AuthController.java
package com.example.jwt.controller;

import com.example.jwt.service.JwtTokenService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.context.annotation.Bean;

@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {

    private final JwtTokenService jwtTokenService;
    private final AuthenticationManager authenticationManager;

    // 학습용 인메모리 사용자
    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        var user = User.builder()
                .username("user")
                .password(passwordEncoder.encode("password"))
                .roles("USER")
                .build();
        return new InMemoryUserDetailsManager(user);
    }

    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@RequestBody LoginRequest loginRequest) {
        // 1. 사용자 인증
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.username(), loginRequest.password())
        );

        // 2. 인증 성공 시 JWT 생성
        String accessToken = jwtTokenService.createAccessToken(authentication.getName());

        return ResponseEntity.ok(new LoginResponse(accessToken));
    }

    // --- DTOs ---
    public record LoginRequest(String username, String password) {}
    public record LoginResponse(String accessToken) {}
}
```

> 💡 `AuthenticationManager`는 `SecurityConfig`에서 Bean으로 등록해야 주입받을 수 있습니다. Spring Boot 3.x에서는 `AuthenticationConfiguration`을 통해 쉽게 가져올 수 있습니다.

## 5. 테스트

Postman과 같은 API 테스트 도구를 사용하여 테스트합니다.

1.  **로그인 요청**

    - `POST` `http://localhost:8080/auth/login`
    - Body (raw, JSON):
      ```json
      {
        "username": "user",
        "password": "password"
      }
      ```
    - **응답**:
      ```json
      {
        "accessToken": "ey..."
      }
      ```

2.  **보호된 리소스 접근 (성공)**

    - `GET` `http://localhost:8080/some-protected-resource`
    - Headers:
      - `Authorization`: `Bearer ey...` (위에서 받은 accessToken)
    - **응답**: (정상 응답)

3.  **보호된 리소스 접근 (실패 - 토큰 없음)**
    - `GET` `http://localhost:8080/some-protected-resource`
    - **응답**: `401 Unauthorized`

이로써 Spring Security와 `oauth2-resource-server`를 이용한 기본적인 JWT 인증 시스템이 완성되었습니다. 다음 장에서는 이 흐름을 더 깊이 이해하기 위해 커스텀 필터를 구현하는 방법을 알아봅니다.
