# 00. REST API 소개

## 1. 왜 REST인가?

- 프런트엔드(React/Vue/모바일)와 백엔드가 분리되면서 서버는 HTML(View) 대신 JSON 데이터 전달에 집중.
- REST는 고정된 표준이 아니라 "리소스를 URI로 표현 + HTTP 메서드로 행위 표현 + 표현(Representation) 교환"이라는 아키텍처 스타일.
- 목표: 단순성(Simple), 무상태성(Stateless), 캐시 가능성(Cacheable), 일관 인터페이스(Uniform Interface), 계층 구조(Layered System).

## 2. 핵심 용어 정리

| 용어           | 의미                                        | 예시                       |
| -------------- | ------------------------------------------- | -------------------------- |
| Resource       | 식별 가능한 대상(명사)                      | /api/users, /api/users/10  |
| Representation | 리소스 상태 표현(JSON 등)                   | {"id":10,"username":"kim"} |
| HTTP Method    | 리소스에 대한 행위                          | GET/POST/PUT/PATCH/DELETE  |
| Stateless      | 각 요청은 독립적으로 처리(서버 세션 상태 X) | JWT, 토큰 기반 인증선호    |
| Idempotent     | 같은 요청 N번 → 동일 상태 보장              | GET, PUT, DELETE           |
| Safe           | 서버 상태 변경 없음                         | GET, HEAD                  |

## 3. @Controller vs @RestController

| 구분               | @Controller                | @RestController    |
| ------------------ | -------------------------- | ------------------ |
| 목적               | View 렌더링 (Thymeleaf 등) | JSON/XML 응답      |
| 반환 처리          | ViewName → ViewResolver    | 객체 → JSON 직렬화 |
| @ResponseBody 필요 | 개별 메서드에 필요         | 클래스 자체가 포괄 |
| 용도               | SSR 페이지                 | REST API           |

> 실무에서는 동일 프로젝트에서 페이지 + API 혼용 시, 페이지 컨트롤러와 API 컨트롤러를 패키지로 분리하는 것을 권장.

## 4. 기본 예제

```java
@RestController
public class HelloApiController {
    @GetMapping("/api/hello")
    public String hello() { return "Hello, REST API!"; }
}
```

## 5. 리소스 설계 원칙(간단)

- 명사 사용: /api/users/1, /api/orders/2024-01-01
- 컬렉션 plural: /api/users
- 행동은 동사 대신 상태 변경으로 표현: /api/users/1/activate (가급적 PUT /api/users/1 {"active":true})
- 필터링/페이징: /api/users?page=0&size=20&sort=createdAt,desc

## 6. 학습 진행 맵

1. 소개 → 2) HTTP 메서드 & 상태코드 → 3) DTO & Validation → 4) 예외 처리 → 5) CORS → 6) 문서화(Swagger) → 7) CRUD 실전 → (선택) 페이징/HATEOAS/버저닝/캐시
