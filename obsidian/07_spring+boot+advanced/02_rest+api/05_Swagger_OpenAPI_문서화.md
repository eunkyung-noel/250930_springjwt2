# 05. Swagger(OpenAPI) 문서화

## 1. OpenAPI & Swagger 관계

- OpenAPI: REST API 명세 표준 (경로, 파라미터, 응답 스키마, 보안 등 정의)
- Swagger: OpenAPI 명세를 작성·시각화·코드생성 하는 도구 모음 (Swagger UI, Editor, Codegen)

## 2. springdoc-openapi 선택 이유

| 항목               | springdoc-openapi | Springfox   |
| ------------------ | ----------------- | ----------- |
| Spring Boot 3 지원 | O                 | X (중단)    |
| OpenAPI 3          | O                 | 제한적      |
| 설정 용이성        | 높은 편           | 구버전 패턴 |

## 3. 의존성 추가 (Gradle)

```groovy
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.6.0'
}
```

실행 후: http://localhost:8080/swagger-ui.html, JSON: /v3/api-docs

## 4. 전역 메타데이터 설정

```java
@Configuration
public class OpenApiConfig {
  @Bean
  public OpenAPI openAPI(){
    Info info = new Info()
       .title("User API")
       .version("v1.0.0")
       .description("사용자 관리 REST API 명세");
    return new OpenAPI().info(info);
  }
}
```

## 5. 컨트롤러 어노테이션

```java
@Tag(name="users", description="사용자 CRUD API")
@RestController
@RequestMapping("/api/users")
public class UserApiController {

  @Operation(summary="사용자 단건 조회", description="ID로 사용자 정보를 반환")
  @ApiResponses({
    @ApiResponse(responseCode="200", description="성공"),
    @ApiResponse(responseCode="404", description="없음")
  })
  @GetMapping("/{id}")
  public ResponseEntity<UserDto.Response> find(@PathVariable Long id){
     return ResponseEntity.ok(service.findUserById(id));
  }
}
```

## 6. DTO 필드 문서화 (선택)

```java
public record UserResponse(
  @Schema(description="고유 식별자", example="100") Long id,
  @Schema(description="사용자 이름", example="alice") String username,
  @Schema(description="이메일", example="alice@example.com") String email
){}
```

## 7. Try it out 테스트 흐름

1. Swagger UI 접속
2. Authorize(보안 스키마 있을 경우)
3. 요청 파라미터/Body 작성 → Execute
4. cURL, 요청/응답 Raw 확인

## 8. 버저닝 전략과 명세

| 전략       | URL                                 | 장점     | 단점                 |
| ---------- | ----------------------------------- | -------- | -------------------- |
| URL Prefix | /api/v1/users                       | 명확     | URI 증가             |
| Header     | Accept: application/vnd.app.v1+json | 클린 URI | 도구 테스트 번거로움 |
| Query      | /api/users?version=1                | 간단     | 캐시 혼동            |

명세 파일(OpenAPI)로 각 버전 별 description/servers 구분 가능.

## 9. 보안 스키마 정의 예시 (Bearer JWT)

```java
@Bean
public OpenAPI openAPI(){
  String schemeName = "bearer-key";
  SecurityScheme scheme = new SecurityScheme()
      .name(schemeName)
      .type(SecurityScheme.Type.HTTP)
      .scheme("bearer")
      .bearerFormat("JWT");
  return new OpenAPI()
      .addSecurityItem(new SecurityRequirement().addList(schemeName))
      .components(new Components().addSecuritySchemes(schemeName, scheme))
      .info(new Info().title("API").version("v1"));
}
```

컨트롤러에서 `@Operation(security = @SecurityRequirement(name = "bearer-key"))` 추가.

## 10. 품질 체크리스트

- [ ] 모든 공개 Endpoint에 summary/description 존재
- [ ] 4xx/5xx 응답 모델 문서화
- [ ] 보안 스키마 정의/테스트
- [ ] Example 값 제공(프론트 Mock 용이)
