# Exercise 07: Java 17로 깔끔한 LLM 클라이언트 구성 (Groq 예시)

- 예제 소스: https://github.com/notlikelion/250806_modern-java/tree/main/src/step1

#자바17 #java17 #HTTP클라이언트 #httpclient #텍스트블록 #textblocks #LLM #Groq #그록 #Reasoning #추론

---

## 🎯 학습 목표

- Java 17의 [[02_Text_Blocks]]와 [[03_HTTP_Client]]를 활용해 LLM 호출 코드를 간결하게 구성
- 텍스트 생성, 음성 합성(TTS), 리저닝 호출을 하나의 추상 인터페이스(LLM)로 통합
- 응답 본문에서 필요한 필드만 안전하게 추출하는 패턴 이해 및 예외 처리 흐름 익히기
- 관련 배경 개념은 [[00_개요_및_LTS]]와 예시 코드는 [[04_Gemini_API_예제]]를 참고

---

## 📋 사전 준비사항

1. JDK: Java 17 이상 설치 및 IDE 설정 — [[../02_java+basic/01_syntax/IntelliJ/00_개요_및_설치/00_IntelliJ_IDEA_개요]] · [[../02_java+basic/01_syntax/IntelliJ/01_초기설정/00_JDK_설정]]
2. 환경 변수: `GROQ_API_KEY` 설정(시크릿 관리 필수)
3. 참고 문서: [[03_HTTP_Client]] · [[02_Text_Blocks]] · [[01_Record]](선택적)

#환경변수 #environmentvariable

---

## 🧭 빠른 참조(클래스/메서드)

- HttpClient.newHttpClient() — 동기/블로킹 HTTP 클라이언트 생성
- HttpRequest.newBuilder().uri(...).headers(...).POST(...).build() — 요청 조립
- HttpResponse.BodyHandlers.ofString()/ofByteArray() — 응답 바디 처리기 선택
- System.getenv("GROQ_API_KEY") — 환경 변수 읽기
- Paths.get(...), Files.writeString(...), Files.write(...) — 파일 저장

#파일IO #fileio #요청 #응답 #request #response

---

## 🗺️ 구성 개요

```mermaid
flowchart LR
  A[사용자 입력] --> B[Application.main]
  B -->|텍스트 생성| C[Groq Chat API]
  B -->|TTS| D[Groq Speech API]
  B -->|Reasoning| E[Groq Chat API]
  C -->|String| F[콘솔/파일]
  D -->|byte[]| F
  E -->|String| F
```

- 애플리케이션은 인터페이스 `ILLM`을 통해 구체 구현 `Groq`를 사용합니다. 필요 시 [[04_Gemini_API_예제]]처럼 다른 제공자도 같은 인터페이스로 교체 가능.

#인터페이스 #interface

---

## 🧩 구현 코드(Java 17)

```java
package step1;

import step1.llm.Groq;
import step1.llm.LLM;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class Application {
    public static void main(String[] args) {
        // 1) 구현체 생성 (환경 변수 검증 포함)
        LLM llm = new Groq();

        // 2) 텍스트 생성 예시
        String prompt1 = "저녁 메뉴 추천해줘. 100자 이내. 결과만.";
        String result1 = llm.generateText(prompt1); // 기본 모델 사용
        System.out.println(result1);

        // 3) 파이프라인(선택): 카테고리별 프롬프트 → 모델 호출 → 후처리
        List<String> menus = List.of("한식", "중식", "양식", "일식");
        menus.stream()
             .parallel() // 필요 시 병렬 처리(네트워크 I/O 특성 고려)
             .map(x -> "%s 하나를 추천받는 프롬프트를 작성해줘.".formatted(x))
             .map(x -> llm.generateText(x, "moonshotai/kimi-k2-instruct"))
             .map(x -> "%s. 결과만, 꾸미는 문법 없이 100자 이내로".formatted(x))
             .map(x -> llm.generateText(x, "meta-llama/llama-4-maverick-17b-128e-instruct"))
             .forEach(System.out::println);

        // 4) 음성 합성(TTS) — byte[]를 파일로 저장
        String prompt2 = "Our time is running out.";
        byte[] wav = llm.changeTextToSpeech(prompt2);
        try {
            Path filename = Paths.get("%s.wav".formatted(System.currentTimeMillis()));
            Files.write(filename, wav);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        // 5) Reasoning — 결과를 마크다운 파일로 저장
        String prompt3 = "클라우드 엔지니어가 되는 방법을 상세히 알려줘";
        String result3 = llm.useReasoning(prompt3, "openai/gpt-oss-120b");
        try {
            Path filename = Paths.get("%s.md".formatted(System.currentTimeMillis()));
            Files.writeString(filename, result3.replace("\\n", "\n"));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

package step1.llm;

// 인터페이스를 구현하려면 이 추상 클래스를 상속받아 구체 메서드를 작성
public abstract class LLM implements ILLM { }

package step1.llm;

public interface ILLM {
    /// Text Generation : https://console.groq.com/docs/text-chat
    String generateText(String prompt);
    // 오버로딩: 모델 지정
    String generateText(String prompt, String model);

    /// Text to Speech : https://console.groq.com/docs/text-to-speech
    byte[] changeTextToSpeech(String prompt);

    /// Reasoning : https://console.groq.com/docs/reasoning
    String useReasoning(String prompt, String model);
}

package step1.llm;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class Groq extends LLM {
    private final String GROQ_API_KEY;

    public Groq() {
        // 생성 시 환경 변수 확인
        GROQ_API_KEY = System.getenv("GROQ_API_KEY");
        if (GROQ_API_KEY == null) {
            throw new RuntimeException("GROQ_API_KEY가 세팅되지 않았습니다");
        }
    }

    // 재사용 가능한 HTTP 클라이언트([[03_HTTP_Client]] 참고)
    private final HttpClient client = HttpClient.newHttpClient();

    // 공통 요청 함수: 채팅(텍스트/리저닝)
    private String useGroq(String url, String prompt, String model) {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(url))
                .headers("Content-Type", "application/json",
                         "Authorization", "Bearer %s".formatted(GROQ_API_KEY))
                .POST(HttpRequest.BodyPublishers.ofString(
                        """
                        {
                          "messages": [
                            { "role": "user", "content": "%s" }
                          ],
                          "model": "%s"
                        }
                        """.formatted(prompt, model)
                ))
                .build();
        try {
            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
            return response.body();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    // 공통 요청 함수: 음성 합성(TTS)
    private byte[] useGroqAudio(String url, String prompt, String model, String template) {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(url))
                .headers("Content-Type", "application/json",
                         "Authorization", "Bearer %s".formatted(GROQ_API_KEY))
                .POST(HttpRequest.BodyPublishers.ofString(template.formatted(prompt, model)))
                .build();
        try {
            HttpResponse<byte[]> response = client.send(request, HttpResponse.BodyHandlers.ofByteArray());
            return response.body();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private final String groqChatURL = "https://api.groq.com/openai/v1/chat/completions";

    @Override
    public String generateText(String prompt) {
        return useGroq(groqChatURL, prompt, "moonshotai/kimi-k2-instruct")
                .split("\"content\":\"")[1]
                .split("\"}")[0]
                .trim();
    }

    @Override
    public String generateText(String prompt, String model) {
        return useGroq(groqChatURL, prompt, model)
                .split("\"content\":\"")[1]
                .split("\"}")[0]
                .trim();
    }

    private final String groqSpeechURL = "https://api.groq.com/openai/v1/audio/speech";

    @Override
    public byte[] changeTextToSpeech(String prompt) {
        return useGroqAudio(groqSpeechURL, prompt, "playai-tts", """
                {
                  "input": "%s",
                  "model": "%s",
                  "voice": "Aaliyah-PlayAI",
                  "response_format": "wav"
                }
                """);
    }

    @Override
    public String useReasoning(String prompt, String model) {
        return useGroq(groqChatURL, prompt, model)
                .split("\"content\":\"")[1]
                .split(",\"reasoning\"")[0]
                .trim();
    }
}
```

---

## 🔧 실행 및 테스트

1. 환경 변수 `GROQ_API_KEY`를 설정합니다(IDE 재시작 필요할 수 있음).
2. `Application.main`을 실행합니다.
   - Reasoning 결과가 `<타임스탬프>.md`로 저장됩니다.
   - TTS 예시는 주석을 해제하고 실행하면 `<타임스탬프>.wav`가 생성됩니다.
3. 필요 시 스트림에 `.parallel()`을 적용/제거하여 호출 특성을 비교해 보세요. 네트워크 I/O 특성상 과도한 병렬은 제한을 유발할 수 있습니다.

#실행 #run #런 #병렬 #parallel #패럴럴

---

## ✅ 요약

- [[02_Text_Blocks]]로 요청 본문 템플릿을 간결하게 유지하고, [[03_HTTP_Client]]로 동기 호출을 단순화했습니다.
- 인터페이스 `ILLM`으로 텍스트/음성/리저닝 호출을 추상화하여 교체 가능성을 확보했습니다.
- 응답은 최소 구문분석으로 필요한 `content`만 추출했으며, 파일 I/O는 표준 `Files` API를 사용했습니다.
- 더 정교한 비동기/제한 제어가 필요하면 [[../02_동시성/03_CompletableFuture]]와 [[../02_동시성/04_Parallel_Stream]]을 검토하세요.
