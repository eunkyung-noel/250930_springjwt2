# Exercise 02.5: 무한 대화형 AI 상담 챗봇
* [[Exercise+02]]

## 🎯 학습 목표

- **무한 반복문**과 **조건부 종료**를 활용한 실시간 대화 시스템 구현
- **배열 기반 대화 기록 관리** 및 **컨텍스트 유지** 기법 학습
- **캐릭터 기반 AI 페르소나** 구현으로 창의적 프롬프트 엔지니어링 체험
- **사용자 입력 기반 흐름 제어**와 **동적 프롬프트 생성** 마스터

## 📋 사전 준비사항

1. **Gemini API 키 발급**: [Google AI Studio](https://aistudio.google.com/apikey)
2. **Java 개발 환경**: [[IntelliJ]] IDE 설치 및 설정
3. **기본 개념 학습**: 아래 참고 문서들을 미리 읽어보세요

---

## 📚 참고 자료

- [[01_조건문]] - if-else, switch 문 사용법
- [[02_반복문]] - while, for 루프와 break/continue (특히 **무한 반복문**)
- [[03_배열]] - 배열 선언, 초기화, 순회 방법
- [[03_입력]] - Scanner를 활용한 사용자 입력 처리

---

## 🚀 1단계: 무한 대화 루프 구조 설정

### 💻 구현하기

```java
import java.util.Scanner;

public class InfiniteChatBot {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // 메인 메뉴 루프
        while (true) {
            System.out.print("원하시는 명령을 입력해주세요. (시작, 종료): ");
            String command = sc.nextLine();

            if (command.equals("시작")) {
                System.out.println("🧑‍⚕️ AI 상담을 시작합니다!");
                // 상담 로직은 다음 단계에서 구현
            } else if (command.equals("종료")) {
                System.out.println("👋 상담을 종료합니다!");
                break;
            } else {
                System.out.println("❌ 잘못된 입력입니다. '시작' 또는 '종료'를 입력해주세요.");
                continue;
            }
        }
    }
}
```

#while문 #if문 #break #continue 

---

## 🚀 2단계: API 설정 및 캐릭터 페르소나 구성

### 💻 구현하기

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

// 기존 코드에 추가...

// API 설정
String GEMINI_API_KEY = System.getenv("GEMINI_API_KEY");
HttpClient client = HttpClient.newHttpClient();
String myModel = "gemini-2.0-flash"; // 더 나은 성능을 위해 flash 모델 사용
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

System.out.println("✅ 피카츄 상담사가 준비되었습니다!");
```

#프롬프트엔지니어링 #gemini #json

---

## 🚀 3단계: 무한 상담 루프 및 대화 기록 시스템

### 💻 구현하기

```java
// 상담 시작 후 실행되는 무한 대화 루프
String[] conversationHistory = new String[20]; // 최대 20개 대화 기록

for (int i = 0; ; i++) { // 무한 반복: 종료 조건 없음
    System.out.print("💬 " + (i + 1) + "번째 질문을 해주세요 (종료하려면 '종료' 입력): ");
    String userQuestion = sc.nextLine();

    // 사용자가 '종료'를 입력하면 상담 종료
    if (userQuestion.equals("종료")) {
        System.out.println("🌟 상담을 종료합니다! 피카피카~");
        break; // 무한 루프 탈출
    }

    // 동적 프롬프트 생성 (첫 질문과 이후 질문 구분)
    String prompt;
    if (i == 0) {
        // 첫 번째 질문: 컨텍스트 없음
        prompt = """
            [%s]에 대해서 50자 이내로 상담하는 대답을 본인이 만화 주인공 피카츄라고 생각하고 해줘.
            결과는 50자를 넘지 않았으면 해. 마크다운 쓰지마.
            """.formatted(userQuestion);
    } else {
        // 이후 질문들: 이전 대화 기록을 컨텍스트로 활용
        String previousConversations = "";
        for (int j = 0; j < i; j++) {
            if (conversationHistory[j] != null) {
                previousConversations += conversationHistory[j] + ", ";
            }
        }

        prompt = """
            [%s]에 대해서 50자 이내로 상담하는 대답을 본인이 만화 주인공 피카츄라고 생각하고 해줘.
            직전에는 [%s] 이러한 대화를 나누었어. 나눈 대화에 대해서는 언급하지 말고 잠재적으로만 반영해줘.
            결과는 50자를 넘지 않았으면 해. 마크다운 쓰지마.
            """.formatted(userQuestion, previousConversations);
    }
```

#for문 #배열 #equals #TextBlock #context

---

## 🚀 4단계: API 호출 및 응답 처리

### 💻 구현하기

```java
    // API 요청 생성 및 전송
    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(GEMINI_URL))
            .headers("Content-Type", "application/json",
                    "X-goog-api-key", GEMINI_API_KEY)
            .POST(HttpRequest.BodyPublishers.ofString(
                    bodyTemplate.formatted(prompt)
            ))
            .build();

    String aiResponse = "";
    try {
        HttpResponse<String> response = client.send(request,
                HttpResponse.BodyHandlers.ofString());
        aiResponse = response.body()
                .split("\"text\": \"")[1]
                .split("}")[0]
                .replace("\\n", "")
                .replace("\"", "")
                .trim();
    } catch (Exception e) {
        System.err.println("❌ API 호출 오류: " + e.getMessage());
        System.out.println("⚠️ 네트워크 문제로 다시 시도해주세요.");
        continue; // 오류 발생 시 다음 반복으로
    }

    System.out.println("⚡ 피카츄: " + aiResponse);

    // 현재 질문을 대화 기록에 저장 (배열 인덱스 관리)
    if (i < conversationHistory.length) {
        conversationHistory[i] = userQuestion;
    }
} // 무한 for 루프 종료 지점
```

#인덱스 #배열 #문자열 

---

## ✅ 최종 완성 코드

모든 단계를 통합한 완성된 무한 대화형 AI 상담 챗봇입니다:

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Scanner;

public class InfiniteChatBot {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // API 설정
        String GEMINI_API_KEY = System.getenv("GEMINI_API_KEY");
        HttpClient client = HttpClient.newHttpClient();
        String myModel = "gemini-2.0-flash";
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

        // 메인 메뉴 루프
        while (true) {
            System.out.print("원하시는 명령을 입력해주세요. (시작, 종료): ");
            String command = sc.nextLine();

            if (command.equals("시작")) {
                System.out.println("🧑‍⚕️ AI 상담을 시작합니다!");
            } else if (command.equals("종료")) {
                System.out.println("👋 프로그램을 종료합니다!");
                break;
            } else {
                System.out.println("❌ 잘못된 입력입니다.");
                continue;
            }

            // 무한 상담 루프
            String[] conversationHistory = new String[20];

            for (int i = 0; ; i++) { // 무한 반복
                System.out.print("💬 " + (i + 1) + "번째 질문을 해주세요 (종료하려면 '종료' 입력): ");
                String userQuestion = sc.nextLine();

                if (userQuestion.equals("종료")) {
                    System.out.println("🌟 상담을 종료합니다! 피카피카~");
                    break;
                }

                // 동적 프롬프트 생성
                String prompt;
                if (i == 0) {
                    prompt = """
                        [%s]에 대해서 50자 이내로 상담하는 대답을 본인이 만화 주인공 피카츄라고 생각하고 해줘.
                        결과는 50자를 넘지 않았으면 해. 마크다운 쓰지마.
                        """.formatted(userQuestion);
                } else {
                    String previousConversations = "";
                    for (int j = 0; j < i; j++) {
                        if (conversationHistory[j] != null) {
                            previousConversations += conversationHistory[j] + ", ";
                        }
                    }

                    prompt = """
                        [%s]에 대해서 50자 이내로 상담하는 대답을 본인이 만화 주인공 피카츄라고 생각하고 해줘.
                        직전에는 [%s] 이러한 대화를 나누었어. 나눈 대화에 대해서는 언급하지 말고 잠재적으로만 반영해줘.
                        결과는 50자를 넘지 않았으면 해. 마크다운 쓰지마.
                        """.formatted(userQuestion, previousConversations);
                }

                // API 호출
                HttpRequest request = HttpRequest.newBuilder()
                        .uri(URI.create(GEMINI_URL))
                        .headers("Content-Type", "application/json",
                                "X-goog-api-key", GEMINI_API_KEY)
                        .POST(HttpRequest.BodyPublishers.ofString(
                                bodyTemplate.formatted(prompt)
                        ))
                        .build();

                String aiResponse = "";
                try {
                    HttpResponse<String> response = client.send(request,
                            HttpResponse.BodyHandlers.ofString());
                    aiResponse = response.body()
                            .split("\"text\": \"")[1]
                            .split("}")[0]
                            .replace("\\n", "")
                            .replace("\"", "")
                            .trim();
                } catch (Exception e) {
                    System.err.println("❌ API 호출 오류: " + e.getMessage());
                    System.out.println("⚠️ 네트워크 문제로 다시 시도해주세요.");
                    continue;
                }

                System.out.println("⚡ 피카츄: " + aiResponse);

                // 대화 기록 저장
                if (i < conversationHistory.length) {
                    conversationHistory[i] = userQuestion;
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
3. **상담 테스트**:
   - "시작" 명령으로 상담 시작
   - 다양한 질문으로 피카츄와 대화
   - "종료" 명령으로 상담 세션 종료
   - 메인 메뉴로 돌아가 다시 상담 시작 가능

---

## 🎯 학습 포인트

### 고급 반복문 활용

- **무한 `for` 루프**: `for (int i = 0; ; i++)` 구조로 종료 조건 없는 반복
- **중첩 무한 루프**: 메뉴 시스템 + 대화 시스템의 이중 구조
- **조건부 `break`**: 사용자 입력에 따른 동적 루프 탈출

### 배열 기반 메모리 관리

- **대화 기록 배열**: `String[] conversationHistory`로 컨텍스트 유지
- **동적 문자열 연결**: 이전 대화들을 하나의 문자열로 조합
- **배열 인덱스 관리**: 오버플로우 방지를 위한 경계 검사

### 실시간 상호작용 시스템

- **즉시 응답**: 사용자 입력 후 바로 AI 응답 생성
- **캐릭터 페르소나**: 피카츄 캐릭터로 일관된 대화 톤 유지
- **오류 복구**: API 오류 발생 시 대화 중단 없이 재시도

### Exercise 02와의 차이점

- **게임 vs 상담**: 스무고개 게임 → 무한 대화형 상담 시스템
- **유한 vs 무한**: 20번 제한 → 무제한 대화 가능
- **목표 지향 vs 자유 대화**: 정답 맞히기 → 자유로운 상담 대화