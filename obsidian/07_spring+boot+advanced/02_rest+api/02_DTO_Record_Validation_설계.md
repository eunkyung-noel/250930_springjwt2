# 02. DTO · Record · Validation 설계

## 1. 왜 Entity를 그대로 반환하면 안되는가?

- 영속성 모델 공개 → 내부 필드 변경이 External Contract(외부 API 스펙) 파괴.
- 민감 정보 노출 위험 (password, internal flag).
- Lazy 연관 로딩 중 순환참조 직렬화 문제.

## 2. DTO 패턴 & 레이어

| 레이어         | 목적                   | 예시                  |
| -------------- | ---------------------- | --------------------- |
| Request DTO    | 입력 검증/의미 명세    | UserDto.CreateRequest |
| Response DTO   | 출력 전용, 표현 안정성 | UserDto.Response      |
| Domain(Entity) | 비즈니스 규칙/영속     | User                  |

## 3. Java record 활용

```java
public class UserDto {
    public record CreateRequest(String username, String email) {
        public User toEntity(){
            return User.builder().username(username).email(email).build();
        }
    }
    public record UpdateRequest(String username, String email) {}
    public record Response(Long id, String username, String email) {
        public static Response from(User user){
            return new Response(user.getId(), user.getUsername(), user.getEmail());
        }
    }
}
```

- 불변, 보일러플레이트 감소, 직렬화 친화.

## 4. Validation (선택 확장)

```groovy
// build.gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

```java
import jakarta.validation.constraints.*;

public record CreateRequest(
    @NotBlank @Size(max=30) String username,
    @Email String email
) {}
```

```java
@PostMapping
public ResponseEntity<Response> create(@Valid @RequestBody CreateRequest req){
    return ResponseEntity.status(HttpStatus.CREATED).body(service.createUser(req));
}
```

- 전역 예외 처리(`MethodArgumentNotValidException`)로 에러 메시지 구조화.

## 5. 컬렉션 변환 스트림 패턴

```java
public List<UserDto.Response> findAllUsers(){
    return repository.findAll().stream()
        .map(UserDto.Response::from)
        .toList();
}
```

## 6. Update 전략

| 전략        | PUT       | PATCH           |
| ----------- | --------- | --------------- |
| 의미        | 전체 교체 | 부분 수정       |
| Null 필드   | 덮어씀    | 무시(로직)      |
| 구현 난이도 | 낮음      | 중간(필드 머지) |

> 단순 CRUD 학습 단계에서는 PUT(전체 교체) 후 고급 단계에서 PATCH 도입.

## 7. 서비스 계층 책임

- 트랜잭션 demarcation
- 엔티티 조회/검증 후 상태 변경(Dirty Checking)
- DTO ↔ Entity 변환 위임(record 내부 팩토리)

## 8. 에러 시나리오 예

| 시나리오        | 예외                            | 대응 상태 |
| --------------- | ------------------------------- | --------- |
| id 미존재       | IllegalArgumentException        | 404       |
| username 중복   | DuplicateKey/Custom             | 409       |
| Validation 실패 | MethodArgumentNotValidException | 400       |

## 9. 예시 Service 일부

```java
@Transactional(readOnly = true)
public class UserService {
    private final UserRepository userRepository;

    @Transactional
    public UserDto.Response createUser(UserDto.CreateRequest dto){
        User saved = userRepository.save(dto.toEntity());
        return UserDto.Response.from(saved);
    }

    public UserDto.Response findUserById(Long id){
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("사용자를 찾을 수 없습니다: "+id));
        return UserDto.Response.from(user);
    }

    @Transactional
    public UserDto.Response updateUser(Long id, UserDto.UpdateRequest dto){
        User user = userRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("사용자를 찾을 수 없습니다: "+id));
        user.update(dto.username(), dto.email());
        return UserDto.Response.from(user);
    }
}
```

## 10. 팁

- Request/Response DTO를 한 파일(UserDto) 내 중첩 record로 그룹 → 네임스페이스 정리.
- API 버전 변경 시 Response DTO만 신규로 만들어 호환 유지.
