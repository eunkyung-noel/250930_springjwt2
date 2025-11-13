# Exercise 01: Gemini API를 활용한 AI 챗봇 구현

## 🎯 학습 목표

- Java 기본 문법을 활용하여 실제 동작하는 AI 챗봇을 완성
- HTTP 통신과 JSON 데이터 처리 경험
- 외부 API 연동 방법 이해

## 📋 사전 준비사항

1. **Gemini API 키 발급**: [Google AI Studio](https://aistudio.google.com/apikey)
2. **Java 개발 환경**: [[IntelliJ]] IDE 설치 및 설정
3. **기본 개념 학습**: 아래 참고 문서들을 미리 읽어보세요

---

## 🚀 1단계: 프로젝트 기본 구조 설정

### 📚 참고 자료

- [[00_프로그램_기본_구조]] - Java 프로그램의 기본 틀 이해
- [[03_입력]] - Scanner를 활용한 사용자 입력 처리
- [[04_변수_데이터타입_리터럴]] - 변수 선언과 데이터 타입

### 💻 구현하기

```java
import java.util.Scanner;

public class GeminiChat {
    public static void main(String[] args) {
        // 사용자 입력 받기
        Scanner sc = new Scanner(System.in);
        System.out.print("질문을 입력해주세요 💬: ");
        String question = sc.nextLine();

        System.out.println("당신의 질문: [" + question + "]");

        // 다음 단계에서 API 통신 로직을 추가할 예정
    }
}
```

---

## 🚀 2단계: HTTP 클라이언트 설정

### 📚 참고 자료

- [[04_변수_데이터타입_리터럴]] - String 변수 활용
- [[02_출력]] - 디버깅을 위한 출력 방법

### 💻 구현하기

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;

// 기존 코드에 추가...

// HTTP 클라이언트 생성
HttpClient client = HttpClient.newHttpClient();

// API 설정
String GEMINI_API_KEY = "여기에_발급받은_API_키_입력"; // 👈 실제 키로 교체 필요
String rule = "100자 이내, 간결하게, 답변만 출력.";

System.out.println("API 클라이언트 준비 완료!");
```

### ⚠️ 보안 주의사항

- API 키는 절대 GitHub 등 공개 저장소에 업로드하지 마세요
- 실제 프로젝트에서는 환경변수나 설정 파일을 사용하세요

---

## 🚀 3단계: HTTP 요청 객체 생성

### 💻 구현하기

```java
// 기존 코드에 추가...

// HTTP 요청 생성 (빌더 패턴 사용)
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent"))
        .headers("Content-Type", "application/json",
                "X-goog-api-key", GEMINI_API_KEY)
        .POST(HttpRequest.BodyPublishers.ofString(
                // Text Block 문법 (""") 사용
                """
                {
                    "contents": [
                      {
                        "parts": [
                          {
                            "text": "%s %s"
                          }
                        ]
                      }
                    ]
                  }
                """.formatted(question, rule) // 동적 문자열 삽입
        ))
        .build();

System.out.println("HTTP 요청 생성 완료!");
```

---

## 🚀 4단계: API 호출 및 응답 처리

### 📚 참고 자료

- [[05_연산자]] - 문자열 연결과 비교 연산
- **예외 처리** - try-catch 구문 (추후 학습 예정)

### 💻 구현하기

```java
import java.net.http.HttpResponse;

// 기존 코드에 추가...

String result = null;
try {
    // API 호출 및 응답 받기
    HttpResponse<String> response = client.send(request,
            HttpResponse.BodyHandlers.ofString());
    result = response.body();

    System.out.println("API 응답 수신 완료!");
    // 디버그용: System.out.println("Raw response: " + result);

} catch (Exception ex) {
    System.err.println("API 호출 중 오류 발생: " + ex.getMessage());
    return; // 프로그램 종료
}
```

---

## 🚀 5단계: JSON 응답 파싱 및 결과 출력

### 📚 참고 자료

- [[04_변수_데이터타입_리터럴]] - String 클래스 메서드 활용
- [[02_출력]] - 최종 결과 출력

### 💻 구현하기

```java
// 기존 코드에 추가...

// JSON에서 실제 답변 텍스트만 추출
try {
    result = result
            .split("\"text\": \"")[1]    // "text": " 이후 부분 선택
            .split("\"")[0]              // 첫 번째 " 이전 부분 선택
            .replace("\\n", "\n")        // 줄바꿈 문자 처리
            .trim();                     // 앞뒤 공백 제거

    // 최종 결과 출력
    System.out.println("\n🤖 AI의 답변: " + result);

} catch (ArrayIndexOutOfBoundsException e) {
    System.err.println("응답 파싱 중 오류 발생. API 응답 형식을 확인하세요.");
    System.err.println("전체 응답: " + result);
}
```

---

## ✅ 최종 완성 코드

모든 단계를 통합한 완성된 코드입니다:

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Scanner;

public class GeminiChat {
    public static void main(String[] args) {
        // 1단계: 사용자 입력
        Scanner sc = new Scanner(System.in);
        System.out.print("질문을 입력해주세요 💬: ");
        String question = sc.nextLine();
        System.out.println("당신의 질문: [" + question + "]");

        // 2단계: HTTP 클라이언트 설정
        HttpClient client = HttpClient.newHttpClient();
        String GEMINI_API_KEY = "여기에_실제_API_키_입력"; // 👈 교체 필요
        String rule = "100자 이내, 간결하게, 답변만 출력.";

        // 3단계: HTTP 요청 생성
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent"))
                .headers("Content-Type", "application/json",
                        "X-goog-api-key", GEMINI_API_KEY)
                .POST(HttpRequest.BodyPublishers.ofString(
                        """
                        {
                            "contents": [
                              {
                                "parts": [
                                  {
                                    "text": "%s %s"
                                  }
                                ]
                              }
                            ]
                          }
                        """.formatted(question, rule)
                ))
                .build();

        // 4단계: API 호출
        String result = null;
        try {
            HttpResponse<String> response = client.send(request,
                    HttpResponse.BodyHandlers.ofString());
            result = response.body();
        } catch (Exception ex) {
            System.err.println("API 호출 오류: " + ex.getMessage());
            return;
        }

        // 5단계: 응답 파싱 및 출력
        try {
            result = result
                    .split("\"text\": \"")[1]
                    .split("\"")[0]
                    .replace("\\n", "\n")
                    .trim();

            System.out.println("\n🤖 AI의 답변: " + result);
        } catch (ArrayIndexOutOfBoundsException e) {
            System.err.println("응답 파싱 오류 발생");
            System.err.println("전체 응답: " + result);
        }
    }
}
```

---

## 🔧 실행 및 테스트

1. **API 키 설정**: `GEMINI_API_KEY` 변수에 실제 발급받은 키를 입력
2. **프로그램 실행**: [[IntelliJ]]에서 Run 버튼 클릭
3. **테스트**: "안녕하세요", "Java란 무엇인가?" 등의 질문으로 동작 확인
