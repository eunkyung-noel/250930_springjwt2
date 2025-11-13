# Maven 의존성 선언과 버전 관리

#메이븐 #maven #의존성선언 #버전 #시맨틱버전 #semanticversioning

---

## 📝 pom.xml 예시

```xml
<dependencies>
  <!-- Google Gemini SDK -->
  <dependency>
    <groupId>com.google.genai</groupId>
    <artifactId>google-genai</artifactId>
    <version>1.10.0</version>
  </dependency>
  <!-- 환경 변수 로더 -->
  <dependency>
    <groupId>io.github.cdimascio</groupId>
    <artifactId>dotenv-java</artifactId>
    <version>3.2.0</version>
  </dependency>
  <!-- Jackson JSON 매퍼 -->
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.19.2</version>
  </dependency>
</dependencies>
```

- 검색 포털: https://mvnrepository.com/
- 예시 링크: https://mvnrepository.com/artifact/com.google.genai/google-genai/1.10.0

---

## 🔢 시맨틱 버전(semver)

- 형식: MAJOR.MINOR.PATCH
  - MAJOR: 호환성 깨질 수 있는 변경(마이그레이션 고려)
  - MINOR: 하위 호환 유지가 일반적이나 제거/변경 가능성 주의
  - PATCH: 버그/보안 수정 → 가급적 최신 권장
- 팀 정책: 보안 패치 신속 반영, MINOR는 테스트 후 적용, MAJOR는 변경 로그 확인 및 이관 계획 수립

---

## ⚠️ 버전 충돌 대응

- `dependency:tree`로 충돌 탐지 → `mvn dependency:tree`
- `dependencyManagement`로 버전 강제
- BOM(Bill of Materials) 사용으로 호환 버전 집합 관리(Spring 등)
