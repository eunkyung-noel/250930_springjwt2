# 06. CRUD 실전 예제 (User API)

## 1. 요구 사항 요약

- 사용자(User) 정보 CRUD REST API.
- JSON 기반, 상태 코드 표준 준수 (201, 200, 204, 404 등).
- DTO 분리 + 전역 예외 처리 + Swagger 문서화 연동.

## 2. build.gradle (발췌)

```groovy
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'org.springframework.boot:spring-boot-starter-validation'
  implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.6.0'
  runtimeOnly 'com.h2database:h2'
}
```

## 3. Entity

```java
@Entity @Table(name="users") @Getter @NoArgsConstructor
public class User {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  @Column(nullable=false, unique=true) private String username;
  @Column(nullable=false) private String email;
  @Builder public User(String username, String email){ this.username=username; this.email=email; }
  public void update(String username, String email){ this.username=username; this.email=email; }
}
```

## 4. DTO (record)

```java
public class UserDto {
  public record CreateRequest(@NotBlank String username, @Email String email){
    public User toEntity(){ return User.builder().username(username).email(email).build(); }
  }
  public record UpdateRequest(String username, String email) {}
  public record Response(Long id, String username, String email){
    public static Response from(User u){ return new Response(u.getId(), u.getUsername(), u.getEmail()); }
  }
}
```

## 5. Repository

```java
public interface UserRepository extends JpaRepository<User, Long> {}
```

## 6. Service

```java
@Service @RequiredArgsConstructor @Transactional(readOnly = true)
public class UserService {
  private final UserRepository repo;
  @Transactional public UserDto.Response create(UserDto.CreateRequest dto){
     return UserDto.Response.from(repo.save(dto.toEntity())); }
  public UserDto.Response findOne(Long id){
     User u = repo.findById(id).orElseThrow(() -> new IllegalArgumentException("사용자 없음: "+id));
     return UserDto.Response.from(u); }
  public List<UserDto.Response> findAll(){
     return repo.findAll().stream().map(UserDto.Response::from).toList(); }
  @Transactional public UserDto.Response update(Long id, UserDto.UpdateRequest dto){
     User u = repo.findById(id).orElseThrow(() -> new IllegalArgumentException("사용자 없음: "+id));
     u.update(dto.username(), dto.email());
     return UserDto.Response.from(u); }
  @Transactional public void delete(Long id){
     if(!repo.existsById(id)) throw new IllegalArgumentException("사용자 없음: "+id);
     repo.deleteById(id); }
}
```

## 7. Controller

```java
@Tag(name="users", description="사용자 CRUD API")
@RestController @RequestMapping("/api/users") @RequiredArgsConstructor
public class UserApiController {
  private final UserService service;

  @Operation(summary="사용자 생성")
  @PostMapping
  public ResponseEntity<UserDto.Response> create(@Valid @RequestBody UserDto.CreateRequest req){
     UserDto.Response saved = service.create(req);
     URI location = URI.create("/api/users/"+saved.id());
     return ResponseEntity.created(location).body(saved);
  }

  @Operation(summary="전체 사용자 조회")
  @GetMapping public ResponseEntity<List<UserDto.Response>> findAll(){
     return ResponseEntity.ok(service.findAll()); }

  @Operation(summary="사용자 단건 조회")
  @GetMapping("/{id}") public ResponseEntity<UserDto.Response> findOne(@PathVariable Long id){
     return ResponseEntity.ok(service.findOne(id)); }

  @Operation(summary="사용자 수정")
  @PutMapping("/{id}") public ResponseEntity<UserDto.Response> update(
     @PathVariable Long id, @RequestBody UserDto.UpdateRequest req){
     return ResponseEntity.ok(service.update(id, req)); }

  @Operation(summary="사용자 삭제")
  @DeleteMapping("/{id}") public ResponseEntity<Void> delete(@PathVariable Long id){
     service.delete(id); return ResponseEntity.noContent().build(); }
}
```

## 8. 전역 예외 처리

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
  @ExceptionHandler(IllegalArgumentException.class)
  public ResponseEntity<Map<String,Object>> handleIllegal(IllegalArgumentException ex){
     Map<String,Object> body = new LinkedHashMap<>();
     body.put("code", "NOT_FOUND");
     body.put("message", ex.getMessage());
     body.put("status", 404);
     body.put("timestamp", Instant.now());
     return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
  }
}
```

## 9. Postman / Swagger 테스트 체크리스트

- [ ] POST /api/users (201 + Location)
- [ ] GET /api/users (200 + 배열)
- [ ] GET /api/users/{id} (존재/미존재 200/404)
- [ ] PUT /api/users/{id} (200 수정 값)
- [ ] DELETE /api/users/{id} (204)

## 10. 확장 아이디어

| 주제      | 방향                                    |
| --------- | --------------------------------------- |
| 페이징    | Spring Data Page<T> → Response DTO 매핑 |
| 정렬/필터 | /api/users?sort=username,asc&email=...  |
| 캐싱      | GET 리스트에 ETag/If-None-Match         |
| 버전관리  | /api/v2/users 새 필드 추가              |
| 감사로그  | 생성/수정 IP, User-Agent 수집           |
