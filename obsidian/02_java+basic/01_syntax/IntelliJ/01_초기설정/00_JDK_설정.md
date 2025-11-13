## JDK 17 & Azul Zulu 설정

### JDK 17 선택 이유

- **Spring Boot 3.x**의 최소 사양은 **JDK 17 LTS**
- 장기 지원(Long Term Support) 버전으로 안정성 보장

#jdk17 #springboot3 #lts

### Azul Zulu 17 장점

- **멀티 플랫폼 호환**: ARM64 (M1/M2)와 x86_64 (AMD EPYC 등) 모두 동일 이미지로 호환
- **Docker Multi‑Arch** 환경에서 편리한 사용
- **다른 옵션**: Temurin, Corretto 등은 OpenJDK Upstream과 기능 동일

#azulzulu #zulu #docker

### JDK 공급업체별 특징

- **안드로이드 개발**: Temurin 권장
- **AWS 환경**: Corretto 권장
- **Mac/Win 크로스 빌드**: Azul Zulu 권장

### JDK 설정 과정

1. 프로젝트 생성 시 **JDK > JDK 다운로드 > 버전 17 > 공급업체: Zulu** 선택
2. 다른 옵션: Corretto, Temurin 중 환경에 맞게 선택

### AI 학습 활용법

- **초심자용 질문**: `초심자도 이해할 수 있게 자세히 설명해줘.`
  - [JDK 설명 예시 1](https://g.co/gemini/share/99e9ed78f123)
- **중학생 수준 설명**: `실생활 비유를 들어서. 중학생도 이해할 수 있게.`
  - [JDK 설명 예시 2](https://g.co/gemini/share/0db9a254ebdc)

#ai #gemini
