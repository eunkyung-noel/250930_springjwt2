# Exercise 02: 제어문을 활용한 스무고개 게임
* [[Exercise+01]]

## 🎯 학습 목표

- Java의 반복문(`while`, `for`)과 조건문(`if-else`)을 실제 프로젝트에 적용
- 배열을 활용한 데이터 저장 및 관리
- Gemini API를 활용한 대화형 게임 구현
- 사용자 입력에 따른 프로그램 흐름 제어

## 📋 사전 준비사항

1. **Gemini API 키 발급**: [Google AI Studio](https://aistudio.google.com/apikey)
2. **Java 개발 환경**: [[IntelliJ]] IDE 설치 및 설정
3. **기본 개념 학습**: 아래 참고 문서들을 미리 읽어보세요

---

## 📚 참고 자료

- [[01_조건문]] - if-else, switch 문 사용법
- [[02_반복문]] - while, for 루프와 break/continue
- [[03_배열]] - 배열 선언, 초기화, 순회 방법
- [[03_입력]] - Scanner를 활용한 사용자 입력 처리

---

## 🚀 1단계: 기본 게임 루프 구조 설정

#java #Scanner #반복문 #조건문 #while문 #if문

### 💻 구현하기

```java
import java.util.Scanner;

public class TwentyQuestions {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // 게임 메인 루프
        while (true) {
            System.out.print("원하시는 명령을 입력해주세요. (시작, 종료): ");
            String command = sc.nextLine();

            if (command.equals("시작")) {
                System.out.println("🎮 스무고개 게임을 시작합니다!");
                // 게임 로직은 다음 단계에서 구현
            } else if (command.equals("종료")) {
                System.out.println("👋 게임을 종료합니다!");
                break;
            } else {
                System.out.println("❌ 잘못된 입력입니다. '시작' 또는 '종료'를 입력해주세요.");
                continue;
            }
        }
    }
}
```

---

## 🚀 2단계: API 클라이언트 및 환경 설정

### 📚 참고 자료

- **API 키 발급**: [Google AI Studio](https://aistudio.google.com/apikey)
- **REST API 문서**: [Gemini API 기본](https://ai.google.dev/gemini-api/docs?hl=ko#rest)
- **모델 목록**: [Gemini 모델 정보](https://ai.google.dev/gemini-api/docs/models?hl=ko)
- **사용량 제한**: [Rate Limits](https://ai.google.dev/gemini-api/docs/rate-limits?hl=ko)

### 💻 구현하기

#gemini #HttpClient

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Random;

// 기존 코드에 추가...

// API 설정
String GEMINI_API_KEY = System.getenv("GEMINI_API_KEY");
HttpClient client = HttpClient.newHttpClient();
String myModel = "gemini-2.0-flash-lite";
String GEMINI_URL = "https://generativelanguage.googleapis.com/v1beta/models/%s:generateContent"
        .formatted(myModel);

// JSON 요청 템플릿
String bodyTemplate = """
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

System.out.println("✅ API 클라이언트 준비 완료!");
```

### ⚠️ 환경변수 설정 방법

#IntelliJ

IntelliJ에서: **현재 파일 > 세로 점 3개 > 매개변수를 사용하여 실행** → 환경 변수에 `GEMINI_API_KEY=발급받은키` 입력

---

## 🚀 3단계: 문제 생성 로직 구현

### 💻 구현하기

#Random

```java
// 게임 시작 시 실행되는 코드
Random rd = new Random();
int answerLength = rd.nextInt(3, 6); // 3~5글자 단어

String prompt = "한글 기준 %d글자 길이의 스무고개용 띄어쓰기 없는 한글 단어 1개를 과정 없이, 마크다운 같은 꾸미는 옵션 없이, 오로지 단어 결과만 출력해줘."
        .formatted(answerLength);

HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create(GEMINI_URL))
        .headers("Content-Type", "application/json",
                "X-goog-api-key", GEMINI_API_KEY)
        .POST(HttpRequest.BodyPublishers.ofString(
                bodyTemplate.formatted(prompt)
        ))
        .build();

String answer = "";
try {
    HttpResponse<String> response = client.send(request,
            HttpResponse.BodyHandlers.ofString());
    answer = response.body()
            .split("\"text\": \"")[1]
            .split("\"")[0]
            .replace("\\n", "")
            .trim();

    System.out.println("💡 글자수는 " + answer.length() + "글자입니다!");
} catch (Exception e) {
    System.err.println("❌ API 호출 오류: " + e.getMessage());
    continue; // 게임을 다시 시작할 수 있도록
}
```

---

## 🚀 4단계: 질문-답변 루프 구현

### 💻 구현하기

```java
// 질문 기록을 위한 배열 선언
String[] questionHistory = new String[20];

// 20번의 질문 기회 제공
for (int i = 0; i < 20; i++) {
    System.out.print("🤔 " + (i + 1) + "번째 질문을 해주세요: ");
    String userQuestion = sc.nextLine();

    // 프롬프트 생성 (첫 번째 질문과 이후 질문 구분)
    String questionPrompt;
    if (i == 0) {
        questionPrompt = "[%s]는 정답인 [%s]에 대한 질문이야. 예/아니오로 먼저 대답한 뒤 부가적인 설명을 해줘. 절대로 정답인 [%s]를 언급하지 마. 과정 없이 결과만 출력해줘."
                .formatted(userQuestion, answer, answer);
    } else {
        // 이전 질문들을 문자열로 연결
        String prevQuestions = "";
        for (int j = 0; j < i; j++) {
            prevQuestions += questionHistory[j] + ", ";
        }

        questionPrompt = """
            [%s]는 정답인 [%s]에 대한 질문이야.
            예/아니오로 먼저 대답한 뒤 부가적인 설명을 해줘.
            이전 질문들: [%s]
            비슷한 질문이면 언급해줘.
            절대로 정답인 [%s]를 언급하지 마. 과정 없이 결과만 출력해줘.
            """.formatted(userQuestion, answer, prevQuestions, answer);
    }

    // AI에게 질문 전송
    HttpRequest questionRequest = HttpRequest.newBuilder()
            .uri(URI.create(GEMINI_URL))
            .headers("Content-Type", "application/json",
                    "X-goog-api-key", GEMINI_API_KEY)
            .POST(HttpRequest.BodyPublishers.ofString(
                    bodyTemplate.formatted(questionPrompt)
            ))
            .build();

    String aiResponse = "";
    try {
        HttpResponse<String> response = client.send(questionRequest,
                HttpResponse.BodyHandlers.ofString());
        aiResponse = response.body()
                .split("\"text\": \"")[1]
                .split("\"")[0]
                .replace("\\n", "\n")
                .replace("\\\"", "\"")
                .trim();
    } catch (Exception e) {
        System.err.println("❌ API 호출 오류: " + e.getMessage());
        continue;
    }

    System.out.println("🤖 AI의 답변: " + aiResponse);

    // 질문을 기록에 저장
    questionHistory[i] = userQuestion;
```

#for-loop #array-usage #string-manipulation

---

## 🚀 5단계: 답변 확인 및 게임 종료 조건

### 💻 구현하기

```java

    // 답변 추측 기회 제공
    System.out.print("💭 예상되는 답을 입력해주세요 (포기하려면 '포기' 입력): ");
    String userGuess = sc.nextLine();

    if (userGuess.equals(answer)) {
        System.out.println("🎉 정답입니다! 축하합니다!");
        break; // 게임 종료
    } else if (userGuess.equals("포기")) {
        System.out.println("😔 아쉽네요! 포기하셨군요.");
        System.out.println("🔍 정답은 '" + answer + "'였습니다!");
        break; // 게임 종료
    } else {
        System.out.println("❌ 틀렸습니다! 다시 질문해보세요.");
    }

    // 20번 모두 실패한 경우
    if (i == 19) {
        System.out.println("😅 20번의 기회를 모두 사용하셨네요!");
        System.out.println("🔍 정답은 '" + answer + "'였습니다!");
    }
} // for 루프 종료
```

---

## ✅ 최종 완성 코드

모든 단계를 통합한 완성된 스무고개 게임입니다:

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Random;
import java.util.Scanner;

public class TwentyQuestions {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // API 설정
        String GEMINI_API_KEY = System.getenv("GEMINI_API_KEY");
        HttpClient client = HttpClient.newHttpClient();
        String myModel = "gemini-2.0-flash-lite";
        String GEMINI_URL = "https://generativelanguage.googleapis.com/v1beta/models/%s:generateContent"
                .formatted(myModel);

        String bodyTemplate = """
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

        Random rd = new Random();

        // 메인 게임 루프
        while (true) {
            System.out.print("원하시는 명령을 입력해주세요. (시작, 종료): ");
            String command = sc.nextLine();

            if (command.equals("시작")) {
                System.out.println("🎮 스무고개 게임을 시작합니다!");
            } else if (command.equals("종료")) {
                System.out.println("👋 게임을 종료합니다!");
                break;
            } else {
                System.out.println("❌ 잘못된 입력입니다.");
                continue;
            }

            // 문제 생성
            int answerLength = rd.nextInt(3, 6);
            String prompt = "한글 기준 %d글자 길이의 스무고개용 띄어쓰기 없는 한글 단어 1개를 과정 없이, 마크다운 같은 꾸미는 옵션 없이, 오로지 단어 결과만 출력해줘."
                    .formatted(answerLength);

            HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create(GEMINI_URL))
                    .headers("Content-Type", "application/json",
                            "X-goog-api-key", GEMINI_API_KEY)
                    .POST(HttpRequest.BodyPublishers.ofString(
                            bodyTemplate.formatted(prompt)
                    ))
                    .build();

            String answer = "";
            try {
                HttpResponse<String> response = client.send(request,
                        HttpResponse.BodyHandlers.ofString());
                answer = response.body()
                        .split("\"text\": \"")[1]
                        .split("\"")[0]
                        .replace("\\n", "")
                        .trim();

                System.out.println("💡 글자수는 " + answer.length() + "글자입니다!");
            } catch (Exception e) {
                System.err.println("❌ API 호출 오류: " + e.getMessage());
                continue;
            }

            // 질문-답변 루프
            String[] questionHistory = new String[20];

            for (int i = 0; i < 20; i++) {
                System.out.print("🤔 " + (i + 1) + "번째 질문을 해주세요: ");
                String userQuestion = sc.nextLine();

                String questionPrompt;
                if (i == 0) {
                    questionPrompt = "[%s]는 정답인 [%s]에 대한 질문이야. 예/아니오로 먼저 대답한 뒤 부가적인 설명을 해줘. 절대로 정답인 [%s]를 언급하지 마. 과정 없이 결과만 출력해줘."
                            .formatted(userQuestion, answer, answer);
                } else {
                    String prevQuestions = "";
                    for (int j = 0; j < i; j++) {
                        prevQuestions += questionHistory[j] + ", ";
                    }

                    questionPrompt = """
                        [%s]는 정답인 [%s]에 대한 질문이야.
                        예/아니오로 먼저 대답한 뒤 부가적인 설명을 해줘.
                        이전 질문들: [%s]
                        비슷한 질문이면 언급해줘.
                        절대로 정답인 [%s]를 언급하지 마. 과정 없이 결과만 출력해줘.
                        """.formatted(userQuestion, answer, prevQuestions, answer);
                }

                HttpRequest questionRequest = HttpRequest.newBuilder()
                        .uri(URI.create(GEMINI_URL))
                        .headers("Content-Type", "application/json",
                                "X-goog-api-key", GEMINI_API_KEY)
                        .POST(HttpRequest.BodyPublishers.ofString(
                                bodyTemplate.formatted(questionPrompt)
                        ))
                        .build();

                String aiResponse = "";
                try {
                    HttpResponse<String> response = client.send(questionRequest,
                            HttpResponse.BodyHandlers.ofString());
                    aiResponse = response.body()
                            .split("\"text\": \"")[1]
                            .split("\"")[0]
                            .replace("\\n", "\n")
                            .replace("\\\"", "\"")
                            .trim();
                } catch (Exception e) {
                    System.err.println("❌ API 호출 오류: " + e.getMessage());
                    continue;
                }

                System.out.println("🤖 AI의 답변: " + aiResponse);
                questionHistory[i] = userQuestion;

                System.out.print("💭 예상되는 답을 입력해주세요 (포기하려면 '포기' 입력): ");
                String userGuess = sc.nextLine();

                if (userGuess.equals(answer)) {
                    System.out.println("🎉 정답입니다! 축하합니다!");
                    break;
                } else if (userGuess.equals("포기")) {
                    System.out.println("😔 아쉽네요! 포기하셨군요.");
                    System.out.println("🔍 정답은 '" + answer + "'였습니다!");
                    break;
                } else {
                    System.out.println("❌ 틀렸습니다! 다시 질문해보세요.");
                }

                if (i == 19) {
                    System.out.println("😅 20번의 기회를 모두 사용하셨네요!");
                    System.out.println("🔍 정답은 '" + answer + "'였습니다!");
                }
            }
        }

        sc.close();
    }
}
```

---

## 🔧 실행 및 테스트

1. **환경변수 설정**: `GEMINI_API_KEY`에 발급받은 API 키 설정
2. **프로그램 실행**: [[IntelliJ]]에서 Run 버튼 클릭
3. **게임 테스트**:
   - "시작" 명령으로 게임 시작
   - 다양한 질문으로 답을 추측해보기
   - "포기" 또는 정답 맞히기로 게임 종료

---

## 🎯 학습 포인트

### 제어문 활용

- **`while(true)` 루프**: 메인 메뉴 시스템 구현
- **`for` 루프**: 20번의 질문 기회 관리
- **`if-else-if` 사다리**: 사용자 입력에 따른 분기 처리
- **`break`와 `continue`**: 루프 제어

### 배열 활용

- **질문 기록 저장**: `String[] questionHistory`
- **배열 순회**: 이전 질문들을 문자열로 연결
