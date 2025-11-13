# 03_DBaaS와 RDBMS 심화

#DBaaS #BaaS #PaaS #SaaS #RDBMS #알디비엠에스 #Supabase #수파베이스 #RDS

---

## 🎯 학습 목표

- DBaaS, BaaS, PaaS, SaaS의 차이를 이해하고 Supabase의 위치를 파악합니다.
- Supabase와 전통적인 RDBMS(AWS RDS)를 비교하여 장단점을 분석합니다.

---

### 클라우드 서비스 모델 (As-a-Service)

- **SaaS (Software as a Service)**: 소프트웨어 완제품을 구독 (예: Google Docs, Airtable)
- **PaaS (Platform as a Service)**: 앱 개발/배포 플랫폼을 제공 (예: Heroku, Render)
- **BaaS (Backend as a Service)**: 인증, DB, 스토리지 등 백엔드 기능을 API로 제공 (예: Firebase, Supabase)
- **DBaaS (Database as a Service)**: DB 설치/운영/백업을 관리해주는 서비스 (예: AWS RDS, Aiven)

```mermaid
graph TD
    subgraph "사용자 관리 범위"
        direction LR
        SaaS["SaaS<br/>(Airtable, Google Docs)"]
        BaaS["BaaS<br/>(Supabase, Firebase)"]
        PaaS["PaaS<br/>(Render, Heroku)"]
        IaaS["IaaS<br/>(AWS EC2, GCP Compute)"]
        OnPrem["On-Premise<br/>(직접 구축)"]
    end

    SaaS -- "가장 적음" --> BaaS
    BaaS --> PaaS
    PaaS --> IaaS
    IaaS --> OnPrem -- "가장 많음" --> style OnPrem fill:#f9f,stroke:#333,stroke-width:2px

    style SaaS fill:#cff,stroke:#333,stroke-width:2px
    style BaaS fill:#cfc,stroke:#333,stroke-width:2px
    style PaaS fill:#ccf,stroke:#333,stroke-width:2px
    style IaaS fill:#ffc,stroke:#333,stroke-width:2px
```

- **Supabase**는 PostgreSQL을 DBaaS 형태로 제공하면서, 인증/스토리지 등 BaaS 기능을 함께 묶은 서비스입니다.

#클라우드서비스 #cloudservice

---

### Supabase vs AWS RDS (PostgreSQL)

| 구분               | Supabase (BaaS + DBaaS)                                                                                                                                                   | AWS RDS (DBaaS)                                                                                                                                                                                      |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **비유**           | **"풀옵션 오피스텔"**                                                                                                                                                     | **"건물 관리 서비스"**                                                                                                                                                                               |
| **핵심 기능**      | PostgreSQL DB + 인증, 스토리지, API 자동 생성                                                                                                                             | 순수 PostgreSQL DB 관리 (설치, 백업, 모니터링)                                                                                                                                                       |
| **장점**           | - **빠른 개발**: 백엔드 기능이 API로 제공되어 즉시 사용 가능<br/>- **통합 환경**: DB, 인증, 파일 저장을 한 곳에서 관리<br/>- **초심자 친화적**: 복잡한 인프라 설정 불필요 | - **유연성/제어**: DB 파라미터, 네트워크(VPC) 등 세부 설정 가능<br/>- **성능/확장성**: 다양한 인스턴스 타입, Multi-AZ 등 고가용성 옵션<br/>- **생태계**: AWS의 다른 서비스(EC2, S3 등)와 긴밀한 연동 |
| **단점**           | - **제한된 제어**: DB 세부 튜닝이나 네트워크 구성에 제약<br/>- **플랫폼 종속성**: Supabase가 제공하는 기능에 의존                                                         | - **초기 설정 복잡**: VPC, 보안 그룹 등 AWS 인프라 지식 필요<br/>- **별도 개발 필요**: 인증, API 등은 직접 EC2/Lambda 등으로 구현해야 함                                                             |
| **주요 사용 사례** | 프로토타입, MVP, 소규모 프로젝트, 프론트엔드 중심 개발                                                                                                                    | 중대규모 서비스, 복잡한 백엔드 로직, 높은 수준의 보안/성능이 요구되는 시스템                                                                                                                         |
| **면접 포인트**    | "빠른 프로토타이핑을 위해 Supabase를 사용해 백엔드 개발 시간을 단축했습니다."                                                                                             | "AWS RDS의 Multi-AZ와 읽기 복제본을 구성하여 DB 고가용성과 부하 분산을 경험했습니다."                                                                                                                |

#면접 #interview #인터뷰 #프로토타이핑 #prototyping #고가용성 #highavailability #HA

---

### RDBMS의 핵심 개념

- **데이터베이스(Database)**: 데이터를 관리하고 호출하는 데 쓰이는 소프트웨어.
- **DBMS (Database Management System)**: 데이터베이스를 관리하는 시스템.
- **RDBMS (Relational DBMS)**: '관계형' 데이터베이스 관리 시스템.
  - **테이블(Table)**: 최소 데이터 묶음의 단위.
  - **행(Row/Tuple)**: 테이블의 각 데이터 항목.
  - **열(Column/Attribute)**: 데이터의 속성.
  - **관계(Relation)**: 테이블들 간의 논리적 연결. 이 관계를 다루는 언어가 **SQL**입니다.
- **NoSQL (Not Only SQL)**: 관계형 모델을 벗어난 DB. (예: Redis - 인메모리, MongoDB - 문서 기반)

#RDBMS #NoSQL #테이블 #table #관계 #relation #릴레이션
