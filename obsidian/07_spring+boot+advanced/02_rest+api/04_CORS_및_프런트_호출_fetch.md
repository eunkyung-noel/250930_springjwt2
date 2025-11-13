# 04. CORS 및 프런트 호출(fetch)

## 1. Same-Origin Policy & CORS

| 항목               | 구성                         | 예                                 |
| ------------------ | ---------------------------- | ---------------------------------- |
| Origin             | Protocol + Host + Port       | http://localhost:8080              |
| 다른 출처 예       | 포트 다름                    | http://localhost:5500              |
| 브라우저 차단 대상 | 스크립트로 cross-origin 요청 | fetch("http://localhost:8080/api") |
| 허용 메커니즘      | 서버 응답 CORS 헤더          | Access-Control-Allow-Origin        |

## 2. Preflight(사전 요청)

- 조건: (메서드가 GET/HEAD/POST 이외, 혹은 POST라도 Content-Type이 application/json 외 커스텀, 커스텀 헤더 Authorization 등)
- 브라우저가 OPTIONS 요청 → CORS 허용 헤더 확인 후 실제 요청 전송.

## 3. @CrossOrigin 단일 컨트롤러

```java
@RestController
@RequestMapping("/api/memos")
@CrossOrigin(origins = "http://localhost:5500")
public class MemoController {
    // ... 동일
}
```

- 빠른 실습용, 다수 컨트롤러/환경 분기 시 전역 설정 권장.

## 4. 전역 CORS 설정

```java
@Configuration
public class WebCorsConfig implements WebMvcConfigurer {
  @Override
  public void addCorsMappings(CorsRegistry registry) {
    registry.addMapping("/api/**")
        .allowedOrigins("https://example.com", "http://localhost:5500")
        .allowedMethods("GET","POST","PUT","PATCH","DELETE")
        .allowedHeaders("Authorization","Content-Type")
        .allowCredentials(true)
        .maxAge(3600);
  }
}
```

## 5. 간단 메모 API (In-Memory)

```java
@RestController
@RequestMapping("/api/memos")
public class MemoController {
    private final ConcurrentHashMap<Long, Memo> store = new ConcurrentHashMap<>();
    private final AtomicLong seq = new AtomicLong();

    @GetMapping public List<Memo> findAll(){ return new ArrayList<>(store.values()); }

    @PostMapping public Memo create(@RequestBody Memo dto){
        long id = seq.incrementAndGet();
        Memo saved = new Memo(id, dto.content());
        store.put(id, saved); return saved;
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id){
        store.remove(id); return ResponseEntity.noContent().build();
    }

    public record Memo(Long id, String content) {}
}
```

## 6. 브라우저 fetch 예시 (Same-Origin)

```html
<script>
  async function load() {
    const res = await fetch("/api/memos");
    const data = await res.json();
    console.log(data);
  }
  load();
</script>
```

## 7. Cross-Origin 재현(JS 별도 호스트)

```js
const API = "http://localhost:8080";
async function load() {
  const res = await fetch(`${API}/api/memos`);
  const list = await res.json();
  console.log(list);
}
```

- 오류: CORS policy: No 'Access-Control-Allow-Origin' header... → 서버 설정 필요.

## 8. 보안 고려 사항

| 위험               | 설명             | 대응                                    |
| ------------------ | ---------------- | --------------------------------------- |
| 와일드카드 \* 남용 | 모든 Origin 허용 | 구체 Origin 목록 유지                   |
| Credentials + \*   | 브라우저 차단    | allowCredentials(true) 시 명시적 Origin |
| 민감 헤더 노출     | Authorization 등 | allowedHeaders 제한                     |

## 9. 정리 체크리스트

- [ ] 개발/운영 Origin 분리
- [ ] Preflight 캐시 maxAge 설정
- [ ] Credentials 필요 여부 명확화 (세션/JWT 쿠키)
- [ ] OPTIONS 200 응답 확인
