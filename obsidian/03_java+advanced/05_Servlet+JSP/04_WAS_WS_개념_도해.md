# WAS/WS 개념 도해

- 배경: [[00_개요_및_로드맵]]

#WAS #웹서버 #webserver #와스 #웹서버 #톰캣 #tomcat

---

## 역할 구분

- Web Server: 정적 파일 서빙, 리버스 프록시/로드밸런싱
- WAS(Tomcat): 서블릿/JSP 실행, 세션/보안/트랜잭션 관리

```mermaid
graph TB
    subgraph "Client Tier"
        C[Client Browser]
    end
    subgraph "Web Tier"
        WS[Web Server\nNginx/Apache]
        subgraph "Static Content"
            HTML[HTML]
            CSS[CSS]
            JS[JavaScript]
            IMG[Images]
        end
    end
    subgraph "Application Tier"
        WAS[Tomcat]
        subgraph "Dynamic Content"
            SERVLET[Servlets]
            JSP[JSP]
        end
    end
    subgraph "Database Tier"
        DB[(Database)]
    end
    C -->|HTTP| WS
    WS -->|Static| HTML
    WS -->|Dynamic| WAS
    WAS --> SERVLET
    WAS --> JSP
    WAS -->|DB Query| DB
    WS -->|Response| C
```

---

## JSP 처리 시퀀스

```mermaid
sequenceDiagram
    participant Client as Client
    participant Tomcat as Tomcat
    participant Jasper as JSP Engine
    participant ServletContainer as Servlet Container

    Client->>Tomcat: GET /hello.jsp
    Tomcat->>Jasper: JSP 처리 요청
    Jasper->>Jasper: JSP → Servlet 변환/컴파일
    Jasper->>ServletContainer: 로드/초기화
    ServletContainer->>Client: HTML 응답
```
