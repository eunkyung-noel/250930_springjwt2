# Exercise 03: OOP로 구조화된 AI 챗봇 리팩터링

#java #oop #인터페이스 #interface #추상 #abstract #오버라이딩 #overriding #다형성 #polymorphism #예외 #exception #throw #throws #HttpClient #gemini #설계 #design

---

https://github.com/notlikelion/250804_oop/tree/main/src/step4

## 🎯 학습 목표

- 인터페이스/추상 클래스로 책임 분리하고 다형성 적용하기
- 오버라이딩 규칙을 지키며 동작을 커스터마이즈하기
- 예외 전파(throws)와 명시적 발생(throw)로 안정적 흐름 제어하기
- HTTP 통신 계층과 프롬프트 구성 계층을 분리해 재사용성 높이기

## 📋 사전 준비사항

1. Gemini API 키: [Google AI Studio](https://aistudio.google.com/apikey)
2. IDE: [[IntelliJ]] 설치 및 실행 설정
3. 사전 학습 문서
   - OOP 기초: [[01_클래스, 객체, 메서드]]
   - 상속/인터페이스: [[02_상속, 인터페이스]]
   - 예외 처리: [[03_예외 처리]]

## 📚 자주 쓰는 키워드/개념 빠른 참조

- `implements`(인터페이스 구현) / `extends`(상속) — [[02_상속, 인터페이스]]
- `@Override`(컴파일러 검사로 안전한 재정의) — [[02_상속, 인터페이스]]
- `throw`/`throws`/`try-catch-finally` — [[03_예외 처리]]
- 접근 제어자/`final`/`static` — [[01_클래스, 객체, 메서드]]
- Java 17, `HttpClient`/`HttpRequest`/`HttpResponse`

---

## 🚀 1단계: 인터페이스로 공통 계약 정의

#인터페이스 #interface #throws

- 목적: “챗봇은 메시지를 받아 문자열을 돌려준다”라는 계약을 타입으로 고정.
- 예외는 호출자가 처리하도록 `throws`로 전파.

```java
// 공통 계약
public interface IGemini { // [[02_상속, 인터페이스]] 참고
    String chat(String message) throws Exception; // [[03_예외 처리]] throws 전파
}
```

---

## 🚀 2단계: 추상 클래스로 공통 구현 제공

#추상 #abstract #HttpClient #super

- 역할: 요청 메시지 가공 → HTTP 호출 → 응답 파싱의 공통 흐름 제공.
- 설계 포인트: `final String apiKey` 보관, 공통 템플릿 메서드 패턴 적용.

```java
public abstract class Chatbot implements IGemini {
    final String apiKey;                   // [[01_클래스, 객체, 메서드]]의 필드/생성자
    protected Chatbot(String apiKey){ this.apiKey = apiKey; }

    @Override
    public String chat(String message) throws Exception {
        String templated = handleMessage(message);      // 프롬프트 템플릿 구성
        String raw = callGemini(apiKey, templated);     // HTTP 통신
        return changeResult(raw);                       // 응답 텍스트만 추출
    }

    private String handleMessage(String message){
        return """
                {
                  \"contents\": [{ \"parts\": [{ \"text\": \"%s\" }]}]
                }
                """.formatted(message);
    }

    private static final String GEMINI_URL =
        "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent";
    private static final java.net.http.HttpClient client = java.net.http.HttpClient.newHttpClient();

    private String callGemini(String apiKey, String body)
            throws java.io.IOException, InterruptedException {
        var request = java.net.http.HttpRequest.newBuilder()
            .uri(java.net.URI.create(GEMINI_URL))
            .headers("Content-Type","application/json","X-goog-api-key", apiKey)
            .POST(java.net.http.HttpRequest.BodyPublishers.ofString(body))
            .build();
        var response = client.send(request, java.net.http.HttpResponse.BodyHandlers.ofString());
        return response.body();
    }

    private String changeResult(String result){
        return result.split("\"text\": \"")[1]
                     .split("}")[0]
                     .replace("\\n", "")
                     .replace("\"", "")
                     .trim();
    }
}
```

---

## 🚀 3단계: 구체 클래스에서 오버라이딩으로 역할 확장

#오버라이딩 #overriding #다형성 #polymorphism

- `RoleChatbot`: 역할(role) 지시문을 덧붙여 감정/톤 등을 제어.
- `EnglishChatbot`: 언어 파라미터로 답변 언어를 제어, 빈 질문은 예외 발생.

```java
public class RoleChatbot extends Chatbot {
    private final String role;
    public RoleChatbot(String apiKey){ super(apiKey); this.role = "너는 위로를 위한 챗봇이야. 200자 이내로 감정적 위로를 위한 내용으로 답변해줘."; }
    public RoleChatbot(String apiKey, String role){ super(apiKey); this.role = role; }
    @Override public String chat(String message) throws Exception {
        if (message.trim().isEmpty()) throw new Exception("빈 질문은 안 됩니다!"); // [[03_예외 처리]] throw
        return super.chat("%s. %s".formatted(message, role));
    }
}

public class EnglishChatbot extends Chatbot {
    private final String language;
    public EnglishChatbot(String apiKey){ super(apiKey); this.language = "영어"; }
    public EnglishChatbot(String apiKey, String language){ super(apiKey); this.language = language; }
    @Override public String chat(String message) throws Exception {
        if (message.trim().isEmpty()) throw new Exception("빈 질문은 안 됩니다!");
        return super.chat("%s. 100자 이내의 평문으로 된 %s(으)로 대답해줘.".formatted(message, language));
    }
}
```

---

## 🚀 4단계: 실행 프로그램에서 전파된 예외 처리

#try-catch #throws #Scanner

- 입력 루프, "종료"로 탈출, 예외 메시지는 사용자에게 가볍게 안내.

```java
import java.util.Scanner;

public class GeminiVer3 {
    public static void main(String[] args) {
        String apiKey = System.getenv("GEMINI_API_KEY");
        Scanner sc = new Scanner(System.in);
        Chatbot chatbot = new EnglishChatbot(apiKey, "일본어"); // 다형성 — 참조는 상위 타입
        while (true) {
            System.out.print("질문을 입력하세요 : ");
            String q = sc.nextLine();
            if (q.equals("종료")) { System.out.println("대화 종료"); return; }
            try {
                String resp = chatbot.chat(q);
                System.out.println(resp);
            } catch (Exception e) {
                System.err.println(e.getMessage()); // [[03_예외 처리]] 로깅/메시지
            }
        }
    }
}
```

---

## ✅ 최종 완성 코드

```java
package step4;

import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Scanner;

// 1) 인터페이스 — 공통 계약 정의
public interface IGemini { String chat(String message) throws Exception; }

// 2) 추상 클래스 — 공통 구현 제공
public abstract class Chatbot implements IGemini {
    final String apiKey;
    protected Chatbot(String apiKey){ this.apiKey = apiKey; }
    @Override public String chat(String message) throws Exception {
        String templated = handleMessage(message);
        String raw = callGemini(apiKey, templated);
        return changeResult(raw);
    }
    private String handleMessage(String message){
        return  """
                {\n  \"contents\": [\n    {\n      \"parts\": [\n        {\n          \"text\": \"%s\"\n        }\n      ]\n    }\n  ]\n}
                """.formatted(message);
    }
    private static final String GEMINI_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent";
    private static final HttpClient client = HttpClient.newHttpClient();
    private String callGemini(String apiKey, String text) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(GEMINI_URL))
                .headers("Content-Type", "application/json", "X-goog-api-key", apiKey)
                .POST(HttpRequest.BodyPublishers.ofString(text))
                .build();
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return response.body();
    }
    private String changeResult(String result){
        return result.split("\"text\": \"")[1]
                .split("}")[0]
                .replace("\\n", "")
                .replace("\"", "")
                .trim();
    }
}

// 3) 구체 클래스 — 역할/언어로 동작 확장
public class RoleChatbot extends Chatbot {
    private final String role;
    public RoleChatbot(String apiKey){ super(apiKey); this.role = "너는 위로를 위한 챗봇이야. 200자 이내로 감정적 위로를 위한 내용으로 답변해줘."; }
    public RoleChatbot(String apiKey, String role){ super(apiKey); this.role = role; }
    @Override public String chat(String message) throws Exception {
        if (message.trim().isEmpty()) throw new Exception("빈 질문은 안 됩니다!");
        return super.chat("%s. %s".formatted(message, role));
    }
}

public class EnglishChatbot extends Chatbot {
    private final String language;
    public EnglishChatbot(String apiKey){ super(apiKey); this.language = "영어"; }
    public EnglishChatbot(String apiKey, String language){ super(apiKey); this.language = language; }
    @Override public String chat(String message) throws Exception {
        if (message.trim().isEmpty()) throw new Exception("빈 질문은 안 됩니다!");
        return super.chat("%s. 100자 이내의 평문으로 된 %s(으)로 대답해줘.".formatted(message, language));
    }
}

// 4) 실행 프로그램 — 예외 처리 및 루프
public class GeminiVer3 {
    public static void main(String[] args) {
        String apiKey = System.getenv("GEMINI_API_KEY");
        Scanner sc = new Scanner(System.in);
        Chatbot chatbot = new EnglishChatbot(apiKey, "일본어");
        while (true) {
            System.out.print("질문을 입력하세요 : ");
            String question = sc.nextLine();
            if (question.equals("종료")) { System.out.println("대화 종료"); return; }
            try {
                String resp = chatbot.chat(question);
                System.out.println(resp);
            } catch (Exception e) {
                System.err.println(e.getMessage());
            }
        }
    }
}
```

---

## 🔧 실행 및 테스트

1. 환경 변수 설정: `GEMINI_API_KEY`에 발급받은 키 입력 — [[IntelliJ]] 참고
2. 실행: `GeminiVer3.main` 실행 → 질문 입력 → "종료"로 종료
3. 커스터마이즈: `RoleChatbot`/`EnglishChatbot` 인스턴스로 교체하여 동작 비교

---

## 🎯 학습 포인트 요약

- 계약/공통/확장 계층을 나누면 테스트/교체/확장이 쉬워진다
- 오버라이딩은 시그니처 동일, 접근 범위는 같거나 넓게 — [[02_상속, 인터페이스]]
- 예외는 “복구 가능 위치”에서 처리, 나머지는 전파 — [[03_예외 처리]]
- 필드/생성자/`final`/`static` 등 기본 문법은 [[01_클래스, 객체, 메서드]] 참고
