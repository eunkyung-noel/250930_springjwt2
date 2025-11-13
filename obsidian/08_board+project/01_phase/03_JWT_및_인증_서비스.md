# 1단계: 인증 및 기본 CRUD

## 3. JWT 및 인증 서비스

JWT(JSON Web Token)를 생성, 파싱, 검증하는 유틸리티 클래스와 Spring Security의 인증 과정에서 실제 사용자 정보를 조회하는 서비스를 구현합니다.

### 1. `JwtUtil` - JWT 유틸리티 클래스

`JwtUtil`은 JWT 관련 모든 로직을 캡슐화하는 클래스입니다. 토큰 생성, 클레임 추출, 유효성 검증 등의 기능을 제공합니다.

- `@Component`: Spring 컨테이너가 이 클래스를 빈으로 관리하도록 합니다.
- `@Value`: `application.yml`에 정의된 설정 값(JWT 비밀키, 만료 시간)을 주입받습니다.
- **`secretKey`**: 주입받은 비밀키 문자열을 `Keys.hmacShaKeyFor()`를 통해 암호학적으로 안전한 `SecretKey` 객체로 변환합니다. 이 키는 토큰의 서명을 생성하고 검증하는 데 사용됩니다.
- **`generateToken()`**: 사용자명, 역할, 토큰 종류(Access/Refresh)를 받아 JWT를 생성합니다.
  - `Jwts.builder()`: JWT 빌더를 시작합니다.
  - `.setSubject(username)`: 토큰의 주체(subject)로 사용자명을 설정합니다.
  - `.claim("role", role)`: 커스텀 클레임으로 사용자의 역할을 추가합니다.
  - `.setIssuedAt()`, `.setExpiration()`: 토큰 발급 시간과 만료 시간을 설정합니다.
  - `.signWith(secretKey)`: `secretKey`를 사용하여 토큰에 서명합니다.
- **`validateToken()`**: 토큰을 파싱하여 유효성을 검증합니다. 파싱 과정에서 예외(만료, 형식 오류, 서명 불일치 등)가 발생하면 유효하지 않은 토큰으로 간주합니다.

```java
// src/main/java/com/example/boardpjt/util/JwtUtil.java

package com.example.boardpjt.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;

@Component
public class JwtUtil {

    private final SecretKey secretKey;
    private final Long accessExpiry;
    private final Long refreshExpiry;

    public JwtUtil(
            @Value("${jwt.secret}") String secret,
            @Value("${jwt.expiry.access}") Long accessExpiry,
            @Value("${jwt.expiry.refresh}") Long refreshExpiry) {
        this.secretKey = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
        this.accessExpiry = accessExpiry;
        this.refreshExpiry = refreshExpiry;
    }

    public String generateToken(String username, String role, boolean isRefresh) {
        long expiry = isRefresh ? refreshExpiry : accessExpiry;
        return Jwts.builder()
                .subject(username)
                .claim("role", role)
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + expiry))
                .signWith(secretKey)
                .compact();
    }

    public Claims getClaims(String token) {
        return Jwts.parser()
                .verifyWith(secretKey)
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }

    public String getUsername(String token) {
        return getClaims(token).getSubject();
    }

    public String getRole(String token) {
        return getClaims(token).get("role", String.class);
    }

    public boolean validateToken(String token) {
        try {
            getClaims(token);
            return true;
        } catch (Exception e) {
            // MalformedJwtException, ExpiredJwtException, etc.
            return false;
        }
    }
}
```

### 2. `CustomUserDetailsService` - 사용자 정보 조회 서비스

`UserDetailsService` 인터페이스는 Spring Security가 사용자 정보를 조회할 때 사용하는 표준 인터페이스입니다. 이 인터페이스를 구현하여, 데이터베이스에서 사용자 정보를 가져와 Spring Security가 이해할 수 있는 `UserDetails` 객체로 변환하는 역할을 합니다.

- `loadUserByUsername(String username)`: Spring Security의 인증 과정(로그인 시도, JWT 필터 검증 등)에서 호출되는 핵심 메서드입니다.
- `userAccountRepository.findByUsername(username)`: `UserAccountRepository`를 통해 데이터베이스에서 사용자 정보를 조회합니다.
- `orElseThrow(() -> new UsernameNotFoundException(...))`: 사용자를 찾지 못하면 `UsernameNotFoundException`을 발생시켜 Spring Security에 인증 실패를 알립니다.
- `User.builder()`: 조회된 `UserAccount` 정보를 바탕으로 Spring Security의 `User` 객체(UserDetails 구현체)를 생성합니다.
  - `username()`: 사용자명을 설정합니다.
  - `password()`: 데이터베이스에 저장된 해시된 비밀번호를 설정합니다.
  - `roles()`: 사용자의 역할을 설정합니다. `role` 문자열에서 "ROLE*" 접두사를 제거하여 전달합니다. (Spring Security는 자동으로 "ROLE*" 접두사를 붙여 처리합니다.)

```java
// src/main/java/com/example/boardpjt/service/CustomUserDetailsService.java

package com.example.boardpjt.service;

import com.example.boardpjt.model.entity.UserAccount;
import com.example.boardpjt.model.repository.UserAccountRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserAccountRepository userAccountRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserAccount userAccount = userAccountRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다: " + username));

        return User.builder()
                .username(userAccount.getUsername())
                .password(userAccount.getPassword())
                .roles(userAccount.getRole().replace("ROLE_", ""))
                .build();
    }
}
```

### 3. `UserAccountService` - 회원 관리 비즈니스 로직

`UserAccountService`는 회원가입, 회원 정보 조회 등 사용자 계정과 관련된 비즈니스 로직을 처리합니다.

- `register()`: 회원가입 요청을 처리합니다.
  - `userAccountRepository.findByUsername(username).isPresent()`: 동일한 사용자명이 이미 존재하는지 확인하여 중복 가입을 방지합니다.
  - `passwordEncoder.encode(password)`: `SecurityConfig`에 빈으로 등록한 `PasswordEncoder`를 사용하여 비밀번호를 해시화합니다.
  - `userAccountRepository.save(newUser)`: 새로운 사용자 정보를 데이터베이스에 저장합니다.

```java
// src/main/java/com/example/boardpjt/service/UserAccountService.java

package com.example.boardpjt.service;

import com.example.boardpjt.model.entity.UserAccount;
import com.example.boardpjt.model.repository.UserAccountRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class UserAccountService {

    private final UserAccountRepository userAccountRepository;
    private final PasswordEncoder passwordEncoder;

    @Transactional
    public UserAccount register(String username, String password) {
        if (userAccountRepository.findByUsername(username).isPresent()) {
            throw new IllegalArgumentException("이미 존재하는 사용자명입니다.");
        }

        UserAccount newUser = new UserAccount();
        newUser.setUsername(username);
        newUser.setPassword(passwordEncoder.encode(password));
        newUser.setRole("ROLE_USER"); // 기본 역할은 USER

        return userAccountRepository.save(newUser);
    }
}
```

이제 JWT 토큰 처리와 사용자 인증의 핵심 로직이 모두 구현되었습니다. 다음 단계에서는 이 서비스들을 사용하여 실제 회원가입 및 로그인 요청을 처리하는 컨트롤러와 뷰를 만듭니다.
