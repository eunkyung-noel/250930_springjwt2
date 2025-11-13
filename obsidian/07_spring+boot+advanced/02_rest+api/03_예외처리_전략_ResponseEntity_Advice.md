# 03. 예외 처리 전략 (ResponseEntity · @ExceptionHandler · @RestControllerAdvice)

## 1. 왜 일관된 예외 응답이 중요한가?

- 프런트엔드 및 외부 소비자가 에러 패턴을 예측 가능하게 함.
- 관찰 가능성(로그/모니터링) 향상 → 공통 필드(code, message, timestamp, path) 기준 지표.

## 2. 세 가지 단계별 접근

| 레벨                              | 특징                     | 장점          | 단점                        | 사용 시기       |
| --------------------------------- | ------------------------ | ------------- | --------------------------- | --------------- |
| 메서드 내부 직접 분기             | if-else + ResponseEntity | 간단          | 중복 증가                   | 학습/프로토타입 |
| @ExceptionHandler (개별 컨트롤러) | 특정 예외 로컬 처리      | 범위 제어     | 다수 컨트롤러 반복          | 전환기          |
| @RestControllerAdvice (전역)      | 중앙 집중                | 일관성 극대화 | 세밀 튜닝 시 조건 분기 필요 | 실무 기본       |

## 3. 에러 응답 DTO 예시

```java
public record ErrorResponse(String code, String message, int status, Instant timestamp) {
    public static ErrorResponse of(String code, String message, HttpStatus status){
        return new ErrorResponse(code, message, status.value(), Instant.now());
    }
}
```

## 4. 전역 예외 처리 기본

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgument(IllegalArgumentException ex){
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse.of("NOT_FOUND", ex.getMessage(), HttpStatus.NOT_FOUND));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex){
        String msg = ex.getBindingResult().getFieldErrors().stream()
            .map(err -> err.getField()+":"+err.getDefaultMessage())
            .findFirst().orElse("Validation error");
        return ResponseEntity.badRequest()
            .body(ErrorResponse.of("INVALID_INPUT", msg, HttpStatus.BAD_REQUEST));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex){
        // log.error("Unhandled exception", ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ErrorResponse.of("INTERNAL_SERVER_ERROR", "서버 내부 오류", HttpStatus.INTERNAL_SERVER_ERROR));
    }
}
```

## 5. 컨트롤러 로컬 예시 비교

```java
@GetMapping("/{id}")
public ResponseEntity<UserDto.Response> find(@PathVariable Long id){
    try { return ResponseEntity.ok(service.findUserById(id)); }
    catch (IllegalArgumentException e){ return ResponseEntity.notFound().build(); }
}
```

→ 전역 Advice 적용 시 try-catch 제거 가능.

## 6. 예외 설계 팁

| 원인             | Custom 예외?                        | 기준           |
| ---------------- | ----------------------------------- | -------------- |
| 단순 존재 여부   | IllegalArgumentException 재사용     | 초기 단순 단계 |
| 도메인 오류 다수 | Custom (e.g. UserNotFoundException) | 의미 명확화    |
| 계층적 코드      | BaseBusinessException 상속          | 큰 규모 서비스 |

## 7. 상태 코드 결정 원칙(요약)

- 존재하지 않음 → 404
- 인증 실패 → 401
- 권한 부족 → 403
- 중복/충돌 → 409
- 검증 실패 → 400
- 서버 내부 처리 실패(알 수 없음) → 500

## 8. Error Code Naming

| 패턴         | 예시                 |
| ------------ | -------------------- |
| 리소스\_액션 | USER_NOT_FOUND       |
| 범주\_상세   | AUTH_INVALID_TOKEN   |
| 비즈니스규칙 | ORDER_LIMIT_EXCEEDED |

## 9. 표준화 후 체크리스트

- [ ] 모든 컨트롤러 try-catch 제거되었나?
- [ ] Validation 실패 메시지 정규화 되었나?
- [ ] 커스텀 예외 Logging 레벨 정의 (warn vs error)
- [ ] 추적 ID(MDC/TraceId) 응답 포함 고려?

## 10. 확장: API 버전별 에러 구조

- /v1 → 필수 필드 최소(code,message)
- /v2 → status,timestamp,path 추가
