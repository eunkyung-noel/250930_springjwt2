# Exercise 06: 병렬 스트림으로 빠른 프롬프트 변환 파이프라인

- https://github.com/notlikelion/250806_concurrency/tree/main/src/step2

#동시성 #concurrency #스레드 #thread #병렬 #parallel #병렬스트림 #parallelstream #함수형 #functional #스트림 #stream #람다 #lambda #HttpClient #예외 #exception #익셉션

---

## 🎯 학습 목표

- 병렬 스트림으로 “주제 → 예문 → 영어 변환” 파이프라인을 선언적으로 구성
- HTTP 호출/파싱 함수 분리로 관심사 분리 및 테스트 용이성 확보
- 동시성 주의점(풀, 블로킹, 예외)을 [[01_Thread]], [[02_ExecutorService_Future]], [[04_Parallel_Stream]]와 연결해 이해

## 📋 사전 준비사항

1. API 키: `GEMINI_API_KEY` 환경 변수 설정
2. 사전 학습(동시성/함수형)
   - 스레드 기초: [[01_Thread]] · 풀/퓨처: [[02_ExecutorService_Future]]
   - Parallel Stream 개요와 주의: [[04_Parallel_Stream]]
   - 람다/메서드 참조: [[../01_함수형+프로그래밍/01_람다와_메서드_참조]]
   - 스트림 API 개요: [[../01_함수형+프로그래밍/02_스트림_API_기본]]

## 📚 빠른 참조

- Stream.parallel() — #병렬스트림 #parallelstream
- HttpClient.newHttpClient() / HttpRequest.newBuilder() / HttpResponse.BodyHandlers.ofString()
- System.getenv("GEMINI_API_KEY") — 환경 변수
- Objects::nonNull, forEach(System.out::println) — [[../01_함수형+프로그래밍/03_reduce_수집_및_forEach]]

---

## 🚀 파이프라인 개요(병렬)

- 입력: 주제 목록(List<String>)
- 단계1: 주제 → 한국어 예문 생성(makeExample)
- 단계2: 예문 → 영어 문장 변환(makeTalk)
- 출력: 콘솔로 결과 출력

```mermaid
flowchart LR
  A[주제 List] -->|parallel stream| B[예문 생성]\nmakeExample
  B --> C[영어 변환]\nmakeTalk
  C --> D[출력]
```

---

## 🧩 구현 코드

```java
package step2;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.List;
import java.util.Objects;

public class FastPrompting {
    public static void main(String[] args) throws Exception {
        long startTime = System.currentTimeMillis();
        // 주제 → 예문 → 영어로 바꾸기 (병렬 스트림)
        List<String> subjects = List.of("날씨", "건강", "취업", "취미", "운동");

        subjects.stream().parallel() // [[04_Parallel_Stream]] 참고
                .map(x -> { // 주제 → 예문
                    try { return makeExample(x); } catch (Exception e) { return null; }
                })
                .filter(Objects::nonNull) // 안전하게 진행
                .map(x -> { // 예문 → 영어
                    try { return makeTalk(x); } catch (Exception e) { return null; }
                })
                .filter(Objects::nonNull)
                .forEach(System.out::println);

        System.out.println("실행시간 = " + (System.currentTimeMillis() - startTime) + "ms");
    }

    // 공용 HTTP 클라이언트 — 블로킹 호출이므로 과도한 병렬화 주의 ([[01_Thread]], [[02_ExecutorService_Future]])
    static final HttpClient client = HttpClient.newHttpClient();
    static final String URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent";
    static final String GEMINI_API_KEY = System.getenv("GEMINI_API_KEY");
    static final String template = """
            {
                "contents": [
                  {
                    "parts": [
                      {
                        "text": "%s"
                      }
                    ]
                  }
                ]
              }
            """;

    // 공통 호출 함수 — 텍스트 입력 → 모델 호출 → 텍스트만 추출
    static String callAPI(String text) throws Exception {
        if (GEMINI_API_KEY == null) {
            System.err.println("오! 환경변수가 없네요!");
            throw new RuntimeException("GEMINI_API_KEY 없음");
        }
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(URL))
                .headers("Content-Type", "application/json", "X-goog-api-key", GEMINI_API_KEY)
                .POST(HttpRequest.BodyPublishers.ofString(template.formatted(text)))
                .build();
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        String body = response.body();
        return body.split("\"text\": \"")[1]
                   .split("}")[0]
                   .replace("\\n", "")
                   .replace("\"", "")
                   .trim();
    }

    static String makeExample(String subject) throws Exception {
        // 한국어 예문 50자 이내, 결과만
        return callAPI("%s(이)라는 주제로 한국어 실생활 문장을 꾸미는 문법(마크다운 등) 없이 평문으로 50자 이내로 작성해주고, 과정없이 문장 결과만 출력해주세요.".formatted(subject));
    }

    static String makeTalk(String example) throws Exception {
        // 한국어 문장을 영어로 변환, 결과만
        return callAPI("%s(이)라는 한국어 문장을 영어 문장으로 바꾸는데 꾸미는 문법(마크다운 등) 없이 평문으로 50자 이내로 작성해주고, 과정없이 문장 결과만 출력해주세요.".formatted(example));
    }
}
```

---

## 🔧 실행 및 테스트

1. 환경 변수 확인: GEMINI*API_KEY 설정 — [[00*개요*및*비유]]
2. 실행: `FastPrompting.main` → 결과 출력/실행시간 확인
3. 변형 실습: 주제 수 늘리기/줄이기, `.parallel()` 제거하여 순차 실행 비교 — [[04_Parallel_Stream]]
4. 주의: 외부 API는 네트워크 지연이 있어, 병렬 개수/호출 빈도에 따라 한계를 맞을 수 있음 → 필요 시 풀/배압 고려 — [[02_ExecutorService_Future]]

---

## 🎯 요약

- 병렬 스트림으로 I/O 중심 파이프라인을 간결하게 작성 가능
- HTTP 블로킹 호출이므로 무한한 병렬화는 금물(서버/클라이언트 리소스 고려)
- 예외는 단계 사이에 `filter(Objects::nonNull)`로 안전하게 걸러 흐름 유지
- 더 복잡한 제어가 필요하면 스레드 풀/CompletableFuture 검토 — [[02_ExecutorService_Future]], [[03_CompletableFuture]]
