# Spring Web MVC 개요

#스프링 #Spring #스프링웹MVC #SpringWebMVC #MVC패턴 #MVC_Pattern #디스패처서블릿 #DispatcherServlet

Spring Web MVC는 웹 애플리케이션 개발을 위한 강력한 프레임워크로, MVC(Model-View-Controller) 아키텍처 패턴을 기반으로 합니다. 이 패턴은 애플리케이션의 역할을 세 가지로 명확히 구분하여 코드의 재사용성과 유지보수성을 높입니다.

---

## 1. MVC 패턴이란?

MVC 패턴은 소프트웨어 디자인 패턴 중 하나로, 애플리케이션을 다음 세 가지 역할로 분리합니다.

- **Model**: 데이터와 비즈니스 로직을 담당합니다. 사용자가 요청한 데이터를 처리하고, 데이터베이스와 상호작용하는 등의 핵심 로직을 포함합니다. (예: `UserService`, `UserDTO`, `UserRepository`)
- **View**: 사용자에게 보여지는 UI(사용자 인터페이스)를 담당합니다. Model로부터 받은 데이터를 화면에 렌더링하여 사용자에게 보여줍니다. (예: JSP, Thymeleaf 파일)
- **Controller**: 사용자의 요청(Request)을 받아 Model과 View를 중개하는 역할을 합니다.
  1.  사용자 요청을 수신합니다.
  2.  요청에 맞는 비즈니스 로직(Model)을 호출합니다.
  3.  Model의 처리 결과를 View에 전달하여 최종 응답(Response)을 생성합니다.

이러한 역할 분리를 통해 각 컴포넌트는 독립적으로 개발 및 테스트될 수 있으며, 이는 애플리케이션의 유연성과 확장성을 크게 향상시킵니다.

---

## 2. Spring Web MVC의 구조와 DispatcherServlet

Spring Web MVC의 중심에는 **`DispatcherServlet`**이라는 핵심 컴포넌트가 있습니다. `DispatcherServlet`은 모든 HTTP 요청을 가장 먼저 받는 **프론트 컨트롤러(Front Controller)** 역할을 하며, 전체 요청 처리 흐름을 제어합니다.

### DispatcherServlet의 동작 흐름

```mermaid
graph TD
    A[Client] -- HTTP Request --> B(DispatcherServlet);
    B -- 1. Request --> C{HandlerMapping};
    C -- 2. Controller Info --> B;
    B -- 3. Call Controller --> D[@Controller];
    D -- 4. Business Logic & Return View Name --> B;
    B -- 5. View Name --> E{ViewResolver};
    E -- 6. View Object --> B;
    B -- 7. Render View --> F[View (JSP/Thymeleaf)];
    F -- 8. HTML Response --> A;

    subgraph "Spring Container"
        B
        C
        D
        E
    end
```

1.  **요청 접수**: 클라이언트로부터 HTTP 요청이 들어오면 `DispatcherServlet`이 가장 먼저 요청을 받습니다.
2.  **핸들러 매핑(HandlerMapping)**: `DispatcherServlet`은 `HandlerMapping`에게 요청 URL에 매핑되는 `Controller`가 무엇인지 문의합니다.
3.  **컨트롤러 호출**: `HandlerMapping`으로부터 컨트롤러 정보를 받은 `DispatcherServlet`은 해당 컨트롤러의 메서드를 호출합니다.
4.  **비즈니스 로직 처리**: `@Controller`는 요청에 맞는 비즈니스 로직을 실행하고, 그 결과 데이터(Model)와 보여줄 뷰(View)의 논리적 이름을 `DispatcherServlet`에 반환합니다.
5.  **뷰 리졸버(ViewResolver)**: `DispatcherServlet`은 컨트롤러가 반환한 뷰의 논리적 이름을 `ViewResolver`에게 전달하여 실제 물리적인 뷰 파일(예: `user/register.jsp`)의 경로를 찾도록 요청합니다.
6.  **뷰 렌더링**: `ViewResolver`로부터 뷰 객체를 받은 `DispatcherServlet`은 해당 뷰에게 모델 데이터를 전달하여 최종 응답 화면(HTML)을 렌더링하도록 요청합니다.
7.  **응답 반환**: 렌더링된 HTML 응답은 `DispatcherServlet`을 거쳐 클라이언트에게 최종적으로 반환됩니다.

이처럼 `DispatcherServlet`은 각 컴포넌트 사이의 중재자 역할을 하며, 개발자는 비즈니스 로직을 담는 `Controller`, `Service`와 화면을 그리는 `View`에만 집중할 수 있게 됩니다.
