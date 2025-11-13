# 00_SQL 엔진 개요 및 비교

#데이터베이스 #database #DB #SQL엔진 #sqlengine #RDBMS #알디비엠에스

---

## 🎯 학습 목표

- SQL 엔진(RDBMS)의 역할과 핵심 기능을 이해합니다.
- MySQL, PostgreSQL 등 주요 SQL 엔진의 차이점과 선택 기준을 파악합니다.

---

### SQL 엔진이란?

- **정의**: 관계형 데이터베이스(RDBMS)의 "두뇌"로서, 저장된 데이터를 **SQL(Structured Query Language)** 명령으로 검색·변경·관리하도록 처리하는 소프트웨어 모듈(파서, 옵티마이저, 실행기)을 말합니다.
- **주요 역할**
  - **SQL 파싱·실행**: `INSERT`, `SELECT` 등 명령을 해석하고 실행 계획 수립
  - **트랜잭션 관리**: ACID 보장을 위해 `COMMIT`, `ROLLBACK` 처리
  - **보안·권한**: 사용자 인증, `GRANT`/`REVOKE`로 객체 접근 제어
  - **쿼리 최적화**: 인덱스 사용 결정, 조인 순서 최적화 등으로 성능 향상
  - **복제·백업 지원**: 로그·스냅샷을 통해 고가용성 보장
- **RDBMS vs SQL 엔진**: RDBMS는 엔진 + 스토리지 + 관리 도구 전체를 포함하며, "SQL 엔진"은 내부 처리 코어를 지칭합니다. 다만 실무에선 MySQL·PostgreSQL 같은 RDBMS 전체를 통칭해 "SQL 엔진"이라 부르는 경우도 많습니다.

#SQL #트랜잭션 #transaction #최적화 #optimization

---

### 주요 SQL 엔진

- **MySQL**: 오라클이 관리하는 대표적 오픈소스 RDBMS, 학습 자료가 풍부합니다. #MySQL #마이에스큐엘
- **PostgreSQL**: 표준 준수·확장 기능(윈도 함수·JSONB)에 강점이 있으며, 복잡한 쿼리나 GIS에서 선호됩니다. #PostgreSQL
- **MariaDB**: MySQL 99% 호환, 커뮤니티 주도 완전 GPL 라이선스입니다. #MariaDB #마리아디비
- **Amazon Aurora**: AWS 제공 MySQL/PostgreSQL 호환 클라우드 DB로, 자동 장애조치·읽기 복제본을 제공합니다. #Aurora #오로라 #AWS

### 주요 차이 요약

- **라이선스**: MySQL(오라클 GPL), MariaDB(GPL), PostgreSQL(PostgreSQL License), Aurora(상용 서비스)
- **JSON 처리**: PostgreSQL(JSONB) vs MySQL/MariaDB(JSON)
- **UPSERT 구문**: PostgreSQL `ON CONFLICT`, MySQL/MariaDB `ON DUPLICATE KEY`
- **확장성**: PostgreSQL(논리·물리 복제), MySQL/MariaDB(GTID·Galera), Aurora(분산 스토리지·Multi-AZ)

#라이선스 #license #JSON #확장성 #scalability

---

### SQL 엔진 선택 기준

| 체크포인트        | 설명                                                                                                                                                                                                                               |
| :---------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **프로젝트 유형** | 트랜잭션 위주 웹 서비스(OLTP)는 MySQL·MariaDB, 분석·GIS·복잡한 쿼리(OLAP)는 PostgreSQL 선호                                                                                                                                        |
| **표준·기능**     | 윈도 함수·CTE·JSONB 등 고급 ANSI SQL 기능 필요 시 PostgreSQL 우위                                                                                                                                                                  |
| **스케일/HA**     | 읽기 복제본만 필요→MySQL, 멀티-리전·분산 스토리지→Aurora, 논리 복제·Sharding 유연성→PostgreSQL                                                                                                                                     |
| **라이선스/비용** | 100% 오픈소스 유지→MariaDB·PostgreSQL, 상용 서포트·AWS 통합→Aurora                                                                                                                                                                 |
| **생태계·도구**   | WordPress·LAMP 스택↔MySQL, 데이터 분석·TimescaleDB 확장↔PostgreSQL                                                                                                                                                                 |
| **팀 경험**       | 기존 개발·운영 인력의 익숙함이 유지보수 비용을 크게 좌우                                                                                                                                                                           |
| **트렌드 지표**   | [JetBrains Dev Ecosystem 2024](https://www.jetbrains.com/ko-kr/lp/devecosystem-2024/) 조사: MySQL 48% · PostgreSQL 43% 사용, [Google Trends](https://trends.google.com/trends/) 5년: PostgreSQL 검색량 상승 지속, MySQL 여전히 1위 |

> **실전 팁**: ORM(JPA·Hibernate 등)을 통해 엔진 세부 문법은 추상화되므로, 실제 운영에서는 버전 마이그레이션(예: MySQL 5.7 → 8.0 → Aurora MySQL) 시 인덱스·Character Set·SQL 모드 변경으로 장애가 생기지 않도록 릴리스 노트·호환 가이드를 꼼꼼히 확인하십시오.

#ORM #JPA #마이그레이션 #migration
