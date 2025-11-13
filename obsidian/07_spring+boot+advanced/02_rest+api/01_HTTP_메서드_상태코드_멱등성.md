# 01. HTTP 메서드 · 상태 코드 · 멱등성

## 1. CRUD ↔ HTTP Method

| CRUD         | HTTP   | Body 사용 | 멱등성   | 캐시 가능 | 비고                           |
| ------------ | ------ | --------- | -------- | --------- | ------------------------------ |
| Create       | POST   | O         | X        | X         | 새 리소스 생성(서버가 ID 결정) |
| Read         | GET    | X         | O (Safe) | O         | 서버 상태 변화 없음            |
| Update(전체) | PUT    | O         | O        | X         | 전체 대체(Representation 교체) |
| Update(부분) | PATCH  | O         | (부분)   | X         | 일부 필드 변경 목적            |
| Delete       | DELETE | (X)       | O        | X         | 결과 상태 = 제거됨             |

> POST는 멱등하지 않아 네트워크 재전송 시 중복 생성 문제 → 클라이언트 측 재시도 로직 주의 혹은 서버에서 중복 키 방지.

## 2. 멱등성(idempotence) 비유

- 엘리베이터 닫힘 버튼: 여러 번 눌러도 결과 동일 (GET/PUT/DELETE)
- ATM 출금: 누를 때마다 잔액 감소 (POST)

## 3. 대표 상태 코드 카테고리

| 코드                      | 의미        | 사용 시점                           |
| ------------------------- | ----------- | ----------------------------------- |
| 200 OK                    | 성공        | GET/PUT/PATCH 일반 성공             |
| 201 Created               | 생성됨      | POST 새 리소스 생성 + Location 헤더 |
| 204 No Content            | 본문 없음   | DELETE, 상태 토글 등                |
| 400 Bad Request           | 잘못된 요청 | 필수 필드 누락/형식 오류            |
| 401 Unauthorized          | 인증 필요   | 토큰 만료/미첨부                    |
| 403 Forbidden             | 권한 없음   | 인증 OK + 권한 부족                 |
| 404 Not Found             | 리소스 없음 | 잘못된 id 조회                      |
| 409 Conflict              | 충돌        | 중복 username, 버전 충돌            |
| 422 Unprocessable Entity  | 의미적 오류 | 포맷은 OK, 도메인 규칙 위반         |
| 500 Internal Server Error | 서버 오류   | 예상하지 못한 예외                  |

## 4. 응답 형태 전략

- 단순 성공: 200 + DTO
- 생성: 201 + Location: /api/users/{id}
- 삭제: 204 (본문 없음)
- 오류: 일관된 에러 바디(JSON) + code + message

```json
{
  "code": "USER_NOT_FOUND",
  "message": "사용자를 찾을 수 없습니다.",
  "status": 404,
  "timestamp": "2025-09-22T12:00:00Z"
}
```

## 5. ResponseEntity 활용

```java
@PostMapping("/api/users")
public ResponseEntity<UserDto.Response> create(@RequestBody UserDto.CreateRequest req){
    UserDto.Response saved = service.createUser(req);
    URI location = URI.create("/api/users/"+saved.id());
    return ResponseEntity.created(location).body(saved); // 201 + Location
}

@GetMapping("/api/users/{id}")
public ResponseEntity<UserDto.Response> find(@PathVariable Long id){
    return ResponseEntity.ok(service.findUserById(id));
}

@DeleteMapping("/api/users/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id){
    service.deleteUser(id);
    return ResponseEntity.noContent().build(); // 204
}
```

## 6. Location 헤더

- 새 리소스 URI 명시 표준
- 컬렉션 POST /api/users → /api/users/{newId}

## 7. 공통 응답 포맷 도입 타이밍

- API 수 10+ 또는 팀 협업 시작 시점부터 early adoption 권장
- 에러코드 사전(Code book) 작성 → 프런트/백 공유
