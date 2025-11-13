# 실전 예제: Groq · Gemini 연동 스니펫

#groq #그록 #gemini #제미나이 #google-genai #dotenv #jackson #httpclient

---

## Groq: Chat Completions(JSON 직렬화)

```java
public record GroqRequestBody(List<Message> messages, String model) {
  public record Message(String role, String content) {}
}

ObjectMapper mapper = new ObjectMapper();
GroqRequestBody body = new GroqRequestBody(
  List.of(new GroqRequestBody.Message("user", prompt)), modelName);
String json = mapper.writeValueAsString(body);

HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create("https://api.groq.com/openai/v1/chat/completions"))
  .headers("Content-Type", "application/json", "Authorization", "Bearer %s".formatted(apiKey))
  .POST(HttpRequest.BodyPublishers.ofString(json))
  .build();
String res = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString()).body();
```

---

## Gemini: Java SDK 사용

```java
// https://github.com/googleapis/java-genai
Client client = Client.builder().apiKey(System.getenv("GEMINI_API_KEY")).build();
GenerateContentResponse response = client.models.generateContent(
  "gemini-2.0-flash", "점심 메뉴 추천해주세요", null);
String text = response.candidates().get().get(0).content().get().text();
```

- SDK를 쓰면 HTTP/JSON 보일러플레이트가 줄어듭니다.
- 텍스트 블록 템플릿과 조합해 프롬프트 구성 시 가독성이 좋아집니다 — [[../03_Java17/02_Text_Blocks]]

---

## Dotenv: 시크릿 로딩

```java
Dotenv dotenv = Dotenv.load();
String apiKey = dotenv.get("GROQ_API_KEY");
```

- 환경 변수 미설정 시 `.env` 파일을 통해 개발환경에서만 로컬로 안전하게 로딩

---

## Naver OpenAPI 예시

```java
String url = "https://openapi.naver.com/v1/search/blog.json";
HttpRequest req = HttpRequest.newBuilder()
  .uri(URI.create(url + "?query=" + URLEncoder.encode(keyword, StandardCharsets.UTF_8)))
  .headers("X-Naver-Client-Id", id, "X-Naver-Client-Secret", secret)
  .build();
String body = HttpClient.newHttpClient().send(req, HttpResponse.BodyHandlers.ofString()).body();
```

- 파라미터 인코딩, 인증 헤더 구조, 응답 파싱 패턴 동일

관련 배경: [[../03_Java17/03_HTTP_Client]] · [[../03_Java17/02_Text_Blocks]]
