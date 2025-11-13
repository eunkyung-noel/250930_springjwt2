# 02 JWT Refresh Token 저장 및 관리

#JWT #RefreshToken #리프레시토큰

## `AuthController` 수정

`RefreshTokenRepository` 대신 `RedisTemplate`을 주입받고, 관련 로직을 Redis 명령어에 맞게 수정합니다. 이전 실습에서 만들었던 `RefreshTokenRepository`는 이제 필요 없으므로 삭제합니다.

```java
// controller/AuthController.java
@RestController
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    // private final RefreshTokenRepository refreshTokenRepository; // 기존 저장소 삭제
    private final RedisTemplate<String, String> redisTemplate; // RedisTemplate 주입
    private final JwtUtil jwtUtil;

    // 로그인 API
    @PostMapping("/login")
    public ResponseEntity<Map<String, String>> login(@RequestBody LoginRequest loginRequest, HttpServletResponse response) {
        // ... (인증 로직은 동일) ...
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword())
        );
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        String username = userDetails.getUsername();
        String role = userDetails.getAuthorities().iterator().next().getAuthority().replace("ROLE_", "");

        String accessToken = jwtUtil.createToken(username, role, "access");
        String refreshToken = jwtUtil.createToken(username, role, "refresh");

        // [변경] Refresh Token을 Redis에 저장 (Key: username, Value: refreshToken)
        // 만료 시간은 JWT의 만료 시간과 동일하게 설정
        long refreshExpirationMs = 604800000L; // 7일 (application.yml 값과 일치시켜야 함)
        redisTemplate.opsForValue().set(
                username,
                refreshToken,
                refreshExpirationMs,
                TimeUnit.MILLISECONDS
        );

        response.addCookie(createCookie("refreshToken", refreshToken));
        return ResponseEntity.ok(Map.of("accessToken", accessToken));
    }

    // 토큰 재발급 API
    @PostMapping("/reissue")
    public ResponseEntity<Map<String, String>> reissue(@CookieValue("refreshToken") String refreshToken, HttpServletResponse response) {
        if (refreshToken == null || jwtUtil.isExpired(refreshToken)) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(Map.of("error", "Refresh token is invalid"));
        }

        String username = jwtUtil.getUsername(refreshToken);

        // [변경] Redis에서 사용자 이름으로 저장된 Refresh Token 조회
        String savedToken = redisTemplate.opsForValue().get(username);

        // Redis에 토큰이 없거나, 전달받은 토큰과 일치하지 않으면 오류
        if (savedToken == null || !savedToken.equals(refreshToken)) {
             return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(Map.of("error", "Refresh token mismatch or expired"));
        }

        String role = jwtUtil.getRole(refreshToken);

        // 새로운 토큰 쌍 발급
        String newAccessToken = jwtUtil.createToken(username, role, "access");
        String newRefreshToken = jwtUtil.createToken(username, role, "refresh");

        // [변경] Redis에 새로운 Refresh Token 저장 및 쿠키 업데이트
        long refreshExpirationMs = 604800000L;
        redisTemplate.opsForValue().set(
                username,
                newRefreshToken,
                refreshExpirationMs,
                TimeUnit.MILLISECONDS
        );
        response.addCookie(createCookie("refreshToken", newRefreshToken));

        return ResponseEntity.ok(Map.of("accessToken", newAccessToken));
    }

    // ... (createCookie 메서드는 동일) ...
    private Cookie createCookie(String key, String value) {
        Cookie cookie = new Cookie(key, value);
        cookie.setMaxAge(60 * 60 * 24 * 7);
        cookie.setHttpOnly(true);
        cookie.setPath("/");
        return cookie;
    }
}
```

이제 애플리케이션을 실행하고 로그인하면, Refresh Token은 더 이상 애플리케이션 메모리가 아닌 외부 Redis 서버에 안전하게 저장되고 관리됩니다. 서버를 재시작하거나 여러 대로 확장하더라도 사용자 로그인 상태는 일관되게 유지됩니다.
