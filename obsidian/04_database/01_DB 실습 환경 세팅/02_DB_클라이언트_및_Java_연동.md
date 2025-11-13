# 02_DB 클라이언트(DBeaver) 및 Java 연동

#DBeaver #디비버 #DB클라이언트 #dbclient #JDBC #자바연동

---

## 🎯 학습 목표

- GUI DB 클라이언트인 DBeaver를 사용하여 데이터베이스에 연결하고 기본 기능을 익힙니다.
- Java 애플리케이션에서 Supabase(PostgreSQL) DB에 REST API를 통해 연동하는 방법을 실습합니다.

---

### DBeaver 자주 쓰는 기능

- 테이블·뷰 트리 탐색 후 드래그&드롭으로 `SELECT *` 자동 완성
- 데이터 편집기 셀 더블클릭 후 값 수정, `Ctrl+S`로 저장 (COMMIT)
- 스키마 우클릭 **ER Diagram** → 이미지·SQL 스크립트로 내보내기
- Dark Theme, Git 플러그인 등 확장 지원

#GUI #ERD

---

### DBeaver로 클라우드 DB 연결하기

1.  **드라이버 선택**: PostgreSQL 또는 MySQL 선택
2.  **정보 입력**: Aiven/Supabase에서 확인한 Host, Port, Database, User, Password 입력
3.  **SSL 설정**: **SSL** 탭에서 `Use SSL` 체크 또는 `sslmode=require` 설정
4.  **연결 테스트**: `Test Connection` 버튼으로 성공 여부 확인

#SSL #에스에스엘 #연결 #connection #커넥션

---

### Java + Supabase 연동 (REST API)

Supabase는 PostgreSQL DB에 직접 접근하는 것 외에, 각 테이블에 대한 REST API를 자동으로 생성해줍니다. Java에서는 이 API를 HTTP 클라이언트로 호출하여 데이터를 조작할 수 있습니다.

- **참고**: [[../../03_java+advanced/03_Java17/03_HTTP_Client|Java 17 HTTP Client]], [[../../03_java+advanced/04_의존성/03_JSON_HTTP_기본과_Jackson|Jackson]]

#### 1. 의존성 추가 (`pom.xml`)

```xml
<!-- JSON 처리를 위한 Jackson -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.1</version>
</dependency>
<!-- .env 파일 로드를 위한 Dotenv -->
<dependency>
    <groupId>io.github.cdimascio</groupId>
    <artifactId>dotenv-java</artifactId>
    <version>3.2.0</version>
</dependency>
```

#Maven #메이븐 #의존성 #dependency #Jackson #잭슨 #Dotenv #닷엔브

#### 2. 데이터 조회 (SELECT)

- **Endpoint**: `https://{...}.supabase.co/rest/v1/RECORD?select=*`
- **Java 코드**:

```java
// 응답 데이터를 매핑할 Record
public record Record(long id, String created_at, String question, String answer) {}

// ... HttpClient 설정 ...
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(SUPABASE_URL + "/rest/v1/RECORD?select=*"))
    .header("apikey", API_KEY)
    .header("Authorization", "Bearer " + API_KEY)
    .build();

HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

// Jackson ObjectMapper로 JSON 배열을 List<Record>로 변환
ObjectMapper mapper = new ObjectMapper();
List<Record> records = mapper.readValue(response.body(), new TypeReference<List<Record>>() {});
```

#HTTPClient #ObjectMapper #TypeReference

#### 3. 데이터 추가 (INSERT)

- **Java 코드**:

```java
// INSERT용 DTO (Data Transfer Object)
public record RecordAddDTO(String question, String answer) {}

// ... HttpClient, ObjectMapper 설정 ...
RecordAddDTO newRecord = new RecordAddDTO("새 질문", "새 답변");
String requestBody = mapper.writeValueAsString(newRecord);

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(SUPABASE_URL + "/rest/v1/RECORD"))
    .header("apikey", API_KEY)
    .header("Authorization", "Bearer " + API_KEY)
    .header("Content-Type", "application/json")
    .POST(BodyPublishers.ofString(requestBody))
    .build();

client.send(request, BodyHandlers.ofString());
```

#DTO #POST #포스트

---

### AI 챗봇과 DB 연동 예시

사용자의 질문과 AI의 답변을 받아 DB에 기록하는 간단한 콘솔 애플리케이션입니다.

```java
// ... (Gemini/Groq 클라이언트, Supabase 클라이언트 초기화)

Scanner sc = new Scanner(System.in);
while (true) {
    System.out.print("질문: ");
    String question = sc.nextLine();
    if (question.equals("종료")) break;

    // 1. AI에게 답변 생성 요청
    String answer = geminiClient.chat(question);

    // 2. 질문과 답변을 DB에 저장
    supabaseClient.addRecord(new RecordAddDTO(question, answer));

    System.out.println("답변: " + answer);
    System.out.println("(DB에 저장 완료)");
}
```

#챗봇 #chatbot #AI #인공지능
