# Docker · Render 배포

- 참고: [[../04_의존성/Exercise+08]] · [[../03_Java17/03_HTTP_Client]]

#도커 #docker #도커 #렌더 #render #렌더 #배포 #deployment

---

## Dockerfile (멀티 스테이지)

```docker
# Maven 빌드 스테이지
FROM maven:3.9.8-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -q -B -DskipTests dependency:go-offline
COPY src ./src
RUN mvn -q -B clean package -DskipTests

# Tomcat 배포 스테이지
FROM tomcat:10.1-jdk17-temurin
COPY --from=builder /app/target/*.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

- 첫 단계에서 의존성 캐싱 → 빠른 재빌드, 두 번째 단계는 실행만 포함.

---

## Render 절차

1. https://render.com 등록 → New Web Service
2. GitHub 저장소 연결 후 Dockerfile 감지 확인
3. 환경변수(Secrets)에 `GOOGLE_API_KEY` 등 추가
4. 배포 완료 후 부여된 도메인 접속 → `/` 또는 `/chat`

#시크릿 #secret #환경변수 #environmentvariable
