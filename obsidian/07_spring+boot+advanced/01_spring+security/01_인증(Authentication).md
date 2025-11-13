# 인증 (Authentication)

인증은 사용자가 누구인지 신원을 확인하는 과정입니다. Spring Security는 이 과정을 체계적으로 처리하기 위해 여러 핵심 컴포넌트를 제공합니다. 사용자가 로그인 폼에 아이디와 비밀번호를 입력하고 '로그인' 버튼을 누르는 순간부터 시작되는 내부 동작을 알아봅니다.

---

## 1. 인증의 핵심 주체: `UserDetails`, `UserDetailsService`

Spring Security는 사용자의 정보를 직접 다루지 않고, `UserDetails`와 `UserDetailsService`라는 두 인터페이스를 통해 애플리케이션의 사용자 데이터와 연동합니다.

### 1.1. `UserDetails` 인터페이스

`UserDetails`는 Spring Security가 사용하는 사용자 정보의 명세입니다. 인증된 사용자의 정보(이름, 비밀번호, 권한 등)를 담는 그릇 역할을 합니다.

- **주요 메서드**:
  - `getUsername()`: 사용자의 이름 (ID) 반환
  - `getPassword()`: 해싱된 비밀번호 반환
  - `getAuthorities()`: 사용자에게 부여된 권한 목록(`GrantedAuthority` 컬렉션) 반환
  - `isAccountNonExpired()`, `isAccountNonLocked()`, `isCredentialsNonExpired()`, `isEnabled()`: 계정의 활성화 상태나 만료 여부 등을 반환

JPA를 사용하는 경우, 보통 **`User` 엔티티 클래스가 `UserDetails` 인터페이스를 직접 구현**하도록 설계하는 것이 일반적입니다.

**`User` 엔티티 구현 예시**:

```java
@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor
public class UserAccount implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false)
    private String password;

    private String role; // "USER", "ADMIN" 등 역할 저장

    // UserDetails 인터페이스의 메서드 구현
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // "ROLE_" 접두사는 hasRole() 메서드와 연동하기 위해 필수적입니다.
        return Collections.singletonList(new SimpleGrantedAuthority("ROLE_" + this.role));
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    // 계정이 만료되지 않았는지 리턴 (true: 만료 안됨)
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    // 계정이 잠겨있지 않은지 리턴 (true: 잠기지 않음)
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    // 비밀번호가 만료되지 않았는지 리턴 (true: 만료 안됨)
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    // 계정이 활성화(사용 가능)인지 리턴 (true: 활성화)
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

### 1.2. `UserDetailsService` 인터페이스

`UserDetailsService`는 사용자 이름(username)을 기반으로 DB 등에서 사용자 정보를 조회하는 역할을 담당합니다.

- **핵심 메서드**: `loadUserByUsername(String username)`
  - 이 메서드는 Spring Security가 인증 과정에서 호출합니다.
  - 파라미터로 전달된 `username`을 이용해 데이터베이스에서 사용자 정보를 찾습니다.
  - 조회된 정보를 `UserDetails` 객체로 만들어 반환합니다.
  - 사용자를 찾지 못하면 `UsernameNotFoundException`을 발생시켜야 합니다.

**`UserDetailsService` 구현 예시**:

```java
@Service
@RequiredArgsConstructor
public class UserService implements UserDetailsService {

    private final UserAccountRepository userAccountRepository;

    /**
     * Spring Security가 로그인 요청을 처리할 때 호출하는 메서드.
     * @param username 로그인 시도하는 사용자의 아이디
     * @return UserDetails 타입의 객체 (Spring Security가 인증에 사용)
     * @throws UsernameNotFoundException 해당 사용자가 DB에 없을 경우 발생
     */
    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // DB에서 username으로 사용자 정보를 조회
        UserAccount userAccount = userAccountRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다: " + username));

        // 조회된 사용자 정보를 바탕으로 Spring Security가 사용하는 User 객체를 생성하여 반환
        // 이 객체는 비밀번호 비교 등 인증 과정에 사용됨
        return new org.springframework.security.core.userdetails.User(
                userAccount.getUsername(),
                userAccount.getPassword(), // DB에 저장된 해시({bcrypt}...)를 그대로 전달
                userAccount.getAuthorities() // 엔티티에 구현된 권한 정보 사용
        );
    }
}
```

---

## 2. `PasswordEncoder`: 안전한 비밀번호 관리

사용자의 비밀번호를 절대 평문으로 저장해서는 안 됩니다. `PasswordEncoder`는 비밀번호를 안전하게 **해싱(Hashing)**하는 역할을 합니다. 해싱은 원본 데이터를 복호화할 수 없는 단방향 암호화 기법입니다.

### 2.1. `DelegatingPasswordEncoder` (권장)

Spring Security 5.0부터 권장되는 `PasswordEncoder` 구현체입니다.

- **동작 방식**: 해싱된 비밀번호 앞에 `{bcrypt}`와 같은 알고리즘 식별자(ID)를 붙여 저장합니다. (예: `{bcrypt}$2a$10$...`)
- **핵심 장점**: 유연성과 확장성. 나중에 더 강력한 해싱 알고리즘(예: scrypt)으로 변경하더라도, 기존 사용자들의 비밀번호를 다시 해싱할 필요 없이 시스템을 원활하게 업그레이드할 수 있습니다.

### 2.2. `BCryptPasswordEncoder`

`bcrypt`는 현재 업계 표준으로 널리 사용되는 강력한 해싱 알고리즘입니다. `DelegatingPasswordEncoder`도 내부적으로 기본값으로 `bcrypt`를 사용합니다.

### 2.3. `PasswordEncoder` 빈 등록

`PasswordEncoder`를 애플리케이션 전역에서 사용하기 위해 `SecurityConfig`에 빈(Bean)으로 등록해야 합니다. 이렇게 등록된 빈은 회원가입 시 비밀번호를 해싱하거나, 로그인 시 입력된 비밀번호와 DB에 저장된 해시를 비교하는 데 사용됩니다.

**`SecurityConfig.java`에 빈 등록 예시**:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    // 비밀번호 암호화를 위한 PasswordEncoder 빈을 등록
    @Bean
    public PasswordEncoder passwordEncoder() {
        // 여러 해싱 알고리즘을 지원하는 위임 기반의 PasswordEncoder (권장)
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    // SecurityFilterChain 설정...
    // ...
}
```

이 세 가지 컴포넌트(`UserDetails`, `UserDetailsService`, `PasswordEncoder`)의 상호작용을 통해 Spring Security는 유연하고 안전한 인증 메커니즘을 완성합니다.
