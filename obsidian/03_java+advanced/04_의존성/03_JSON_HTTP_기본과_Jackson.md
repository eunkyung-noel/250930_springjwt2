# JSON · HTTP 기본과 Jackson 연동

#json #HTTP #파싱 #parsing #자바17 #java17

---

## 🧠 핵심 개념

- JSON: 키-값, 객체, 배열로 구성된 텍스트 데이터 포맷. 2020년대 표준 교환 포맷
- HTTP: 인터넷에서 요청/응답을 주고받는 규약. Java 17의 [[../03_Java17/03_HTTP_Client]]로 쉽게 호출 가능
- 파싱: 문자열(JSON) → 객체(Java Record/Class)로 변환하는 과정

---

## 🔡 JSON 예시와 매핑

```json
{
  "key": "value",
  "obj": { "key": "value" },
  "arr": [1, 2, 3]
}
```

- 문자열 다루기(split/정규표현식)로도 가능하지만, 유지보수성과 안정성을 위해 Jackson 같은 파서를 권장
- Jackson(ObjectMapper):
  - writeValueAsString(obj): 객체 → JSON 문자열
  - readValue(json, Type.class): JSON → 객체

---

## 🧩 Java 17 + HttpClient + Jackson

```java
HttpClient http = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder()
        .uri(URI.create("https://api.example.com"))
        .build();
HttpResponse<String> res = http.send(req, HttpResponse.BodyHandlers.ofString());
String json = res.body();
ObjectMapper mapper = new ObjectMapper();
MyDto dto = mapper.readValue(json, MyDto.class);
```

관련 배경: [[../03_Java17/02_Text_Blocks]] · [[../03_Java17/03_HTTP_Client]]
