# Exercise 05: 함수형 스트림으로 AI 이미지 생성 파이프라인 만들기

#함수형 #functional #람다 #lambda #스트림 #stream #메서드참조 #HttpClient #Base64 #파일IO #fileio

---

https://github.com/notlikelion/250805_fp/tree/main/src/step2

## 🎯 학습 목표

- 람다/스트림으로 데이터 수집→프롬프트 생성→이미지 생성 파이프라인 구성
- 불변 리스트·메서드 참조로 선언적/가독성 높은 코드 작성
- 예외/리소스 처리와 HTTP 호출 계층을 명확히 분리
- OOP 버전과의 차이를 비교해 함수형 접근의 강점 이해

## 📋 사전 준비사항

1. API 키: GEMINI\*API_KEY 환경 변수 설정
2. 사전 학습(함수형)
   - 람다/메서드 참조: [[01_람다와_메서드_참조]]
   - 스트림 기본/중간·최종 연산: [[02_스트림_API_기본]]
   - 수집/forEach/리듀스: [[03_reduce_수집_및_forEach]]
   - 전체 맥락: [[04_실습_예제]]

## 📚 자주 쓰는 클래스/메서드 빠른 참조

- HttpClient.newHttpClient(), HttpRequest.newBuilder(), HttpResponse.BodyHandlers.ofString() — HTTP 통신
- Base64.getDecoder().decode(String) — #Base64 #베이스식스티포
- IntStream.range(시작,끝).boxed() — #스트림 #stream #스트림
- Files.write(Path, byte[]) · Paths.get(String) — 파일 저장
- Scanner.nextLine() — #입력 #input #인풋
- System.getenv("GEMINI_API_KEY") — 환경 변수 읽기

---

## 🚀 단계 1: 입력 수집(명령형)

#입력 #input #Scanner #스캐너

- 실생활 비유: 설문지를 n장 받아 목록으로 정리.
- CS 관점: 가변 리스트(List)에 유효성 검사를 거쳐 순차 추가.

```java
package step2;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class ImageGen { // 명령형 버전
    final private List<String> favoriteList = new ArrayList<>();
    final private Scanner scanner = new Scanner(System.in);
    final private int size;
    public ImageGen(int size) { this.size = size; }

    // n번 입력 받아 리스트에 저장
    void inputData() {
        for (int i = 0; i < size; i++) {
            System.out.print("좋아하는 동물을 입력해주세요 : ");
            String input = scanner.nextLine();
            if (input.trim().isEmpty()) { // 공백 방지
                System.out.println("제대로 입력해주세요!");
                i--; // 횟수 유지
                continue;
            }
            favoriteList.add(input);
        }
        System.out.println(favoriteList);
    }
    // ...뒤 단계에서 사용할 필드/메서드가 이어집니다
}
```

---

## 🚀 단계 2: 프롬프트 생성(HTTP 호출 분리)

#HttpClient #예외 #exception #익셉션

- 실생활 비유: 키워드(선호 목록)를 문장으로 다듬는 카피라이터.
- CS 관점: HTTP 호출을 메서드로 캡슐화해 재사용/테스트 용이.

```java
package step2;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.ArrayList;

public class ImageGen {
    // ...existing fields...
    private final HttpClient httpClient = HttpClient.newHttpClient();
    private final List<String> imagePromptList = new ArrayList<>();
    private final String GEMINI_API_KEY = System.getenv("GEMINI_API_KEY");

    private String callAPI(String url, String body) {
        if (GEMINI_API_KEY == null) throw new RuntimeException("GEMINI_API_KEY가 없습니다!");
        HttpRequest req = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .headers("Content-Type","application/json","X-goog-api-key",GEMINI_API_KEY)
            .POST(HttpRequest.BodyPublishers.ofString(body))
            .build();
        try {
            HttpResponse<String> res = httpClient.send(req, HttpResponse.BodyHandlers.ofString());
            return res.body();
        } catch (Exception e) {
            System.err.println(e.getMessage());
            throw new RuntimeException(e);
        }
    }

    void makeImagePrompt() {
        String url = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent";
        for (String v : favoriteList) {
            String result = callAPI(url, """
                {
                  "contents": [{
                    "parts": [{"text": "%s(을)를 이미지로 나타내기 위한 200자 이내의 상세한 프롬프트를 작성해줘. 결과만 작성해줘."}]
                  }]
                }
            """.formatted(v));
            String prompt = result.split("\"text\": \"")[1]
                                   .split("}")[0]
                                   .replace("\\n", "")
                                   .replace("\"", "")
                                   .trim();
            imagePromptList.add(prompt);
        }
        imagePromptList.forEach(System.out::println); // [[03_reduce_수집_및_forEach]]
    }
}
```

---

## 🚀 단계 3: 이미지 생성과 파일 저장

#Base64 #파일IO #fileio

- 실생활 비유: 확정된 문구(프롬프트)로 인쇄소에 주문하고 결과물을 파일로 보관.
- CS 관점: 텍스트(베이스64) → 바이트 디코딩 → 파일 시스템에 저장.

```java
package step2;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Base64;

public class ImageGen {
    // ...existing fields/methods...
    void generateImage() {
        String model = "gemini-2.0-flash-preview-image-generation";
        String url = "https://generativelanguage.googleapis.com/v1beta/models/%s:generateContent".formatted(model);
        for (String prompt : imagePromptList) {
            String result = callAPI(url, """
                {
                  "contents": [{
                    "role": "user",
                    "parts": [{ "text": "%s" }]
                  }],
                  "generationConfig": { "responseModalities": ["IMAGE","TEXT"] }
                }
            """.formatted(prompt));

            String image64 = result.split("\"data\": \"")[1]
                                   .split("}")[0]
                                   .replace("\\n", "")
                                   .replace("\"", "")
                                   .trim();
            byte[] imageBytes = Base64.getDecoder().decode(image64);
            String outputPath = "%s.png".formatted(System.currentTimeMillis());
            Path filePath = Paths.get(outputPath);
            try { Files.write(filePath, imageBytes); }
            catch (Exception e) { System.err.println(e.getMessage()); }
        }
    }
}
```

---

## 🚀 단계 4: 스트림으로 파이프라인 선언하기(함수형)

#스트림 #stream #람다 #lambda #메서드참조

- 실생활 비유: 컨베이어 벨트 위에서 입력→가공→출력을 단계별로 흘려보내기.
- CS 관점: 중간 연산(map/filter)과 최종 연산(forEach)로 선언적 파이프라인 구성 — [[02_스트림_API_기본]]

```java
package step2;

import java.net.URI;
import java.net.http.*;
import java.nio.file.*;
import java.util.Base64;
import java.util.Scanner;
import java.util.stream.IntStream;

public class ImageGenStreamClean { // 함수형 버전
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int size = 3;
        HttpClient http = HttpClient.newHttpClient();
        String urlTpl = "https://generativelanguage.googleapis.com/v1beta/models/%s:generateContent";
        String promptTpl = """
            {"contents":[{"parts":[{"text":"%s(을)를 이미지로 나타내기 위한 200자 이내의 상세한 프롬프트를 작성해줘. 결과만 작성해줘."}]}]}
        """;
        String imageTpl = """
            {"contents":[{"role":"user","parts":[{"text":"%s"}]}],"generationConfig":{"responseModalities":["IMAGE","TEXT"]}}
        """;

        IntStream.range(0, size)
            .boxed()
            .map(i -> { System.out.print("좋아하는 캐릭터는? : "); return sc.nextLine(); })
            .map(x -> send(http, urlTpl.formatted("gemini-2.0-flash"), promptTpl.formatted(x)))
            .filter(res -> res != null)
            .map(ImageGenStreamClean::extractText)
            .map(p -> send(http, urlTpl.formatted("gemini-2.0-flash-preview-image-generation"), imageTpl.formatted(p)))
            .filter(res -> res != null)
            .map(ImageGenStreamClean::extractImage)
            .forEach(ImageGenStreamClean::saveImage); // 메서드 참조 [[01_람다와_메서드_참조]]
    }

    static String send(HttpClient http, String url, String body) {
        try {
            return http.send(
                    HttpRequest.newBuilder()
                        .uri(URI.create(url))
                        .headers("Content-Type","application/json","X-goog-api-key",System.getenv("GEMINI_API_KEY"))
                        .POST(HttpRequest.BodyPublishers.ofString(body)).build(),
                    HttpResponse.BodyHandlers.ofString()
            ).body();
        } catch (Exception e) { System.err.println(e.getMessage()); return null; }
    }
    static String extractText(String json){
        return json.split("\"text\": \"")[1].split("}")[0].replace("\\n","" ).replace("\"","" ).trim();
    }
    static String extractImage(String json){
        return json.split("\"data\": \"")[1].split("}")[0].replace("\\n","" ).replace("\"","" ).trim();
    }
    static void saveImage(String base64){
        try {
            byte[] bytes = Base64.getDecoder().decode(base64);
            Path p = Paths.get("%s.png".formatted(System.currentTimeMillis()));
            Files.write(p, bytes);
        } catch (Exception e) { System.err.println(e.getMessage()); }
    }
}
```

---

## 🔧 실행 및 테스트

1. 환경 변수 확인: GEMINI\*API_KEY가 설정되어 있어야 합니다
2. 명령형 실행: `ImageGen.main` → 입력 → `makeImagePrompt()` → `generateImage()` 순으로 파일 생성 확인
3. 함수형 실행: `ImageGenStreamClean.main` → n회 입력 → PNG 파일 생성 확인
4. 장애 테스트: 환경 변수 미설정, 빈 입력, 네트워크 오류 시 예외 메시지 확인 — [[03_reduce_수집_및_forEach]]의 forEach 예외 주의

---

## 🎯 학습 포인트 요약

- 데이터 흐름을 스트림으로 선언하면 입력→가공→출력 단계가 명료해진다 — [[02_스트림_API_기본]]
- 메서드 참조로 의도를 드러내고 재사용 가능한 순수 함수로 분리 — [[01_람다와_메서드_참조]]
- HTTP 호출/파싱/저장은 관심사를 나눠 테스트 용이성↑
- 명령형 vs 함수형을 비교하며 팀/문맥에 맞는 스타일을 선택
