# OpenMetadata vs DataHub 비교 문서

## 목차
1. 개요
2. 프로젝트 개요 비교
   1. OpenMetadata란?
   2. DataHub란?
3. 아키텍처 및 설계 철학
   1. OpenMetadata 아키텍처
   2. DataHub 아키텍처
4. 핵심 기능 비교
   1. 공통 기능
   2. OpenMetadata 강점 영역
   3. DataHub 강점 영역
5. 기술 스택 및 구성 요소
6. 커넥터·통합(Integrations) 비교
   1. DB / DWH / Data Lake
   2. BI / Dashboard
   3. 파이프라인·오케스트레이션
   4. 스트리밍 / 메시징
   5. ML·Feature Store·기타
7. 배포·운영 관점 비교
8. 라이선스·커뮤니티·성숙도
9. 선택 가이드 (어떤 상황에서 무엇을 쓸까?)
10. 결론 요약
11. 참고 링크
---

## 1. 개요

이 문서는 대표적인 오픈소스 메타데이터 플랫폼인 **OpenMetadata**와 **DataHub**를 기술·기능 관점에서 비교하기 위한 것이다.  
두 프로젝트 모두 데이터 카탈로그, 메타데이터 관리, 데이터 거버넌스·디스커버리·라인리지를 제공하며, 현대적인 데이터·AI 스택을 대상으로 한다.

---

## 2. 프로젝트 개요 비교

### 2.1 OpenMetadata란?

- 공식 사이트에서 “**Open and unified metadata platform for data discovery, observability, and governance**”로 정의되며,  
  단일 메타데이터 그래프 위에서 데이터 디스커버리, 품질, 거버넌스, 협업을 통합 제공하는 플랫폼이다. :contentReference
- GitHub 설명에서도 “데이터 디스커버리, 데이터 옵저버빌리티, 데이터 거버넌스를 위한 통합 메타데이터 플랫폼”으로 소개된다. 
- 다양한 데이터 웨어하우스, DB, 대시보드, 메시징, 파이프라인, ML 서비스 등에 대해 약 80개 이상의 커넥터를 제공한다. 

### 2.2 DataHub란?

- LinkedIn에서 시작된 오픈소스 메타데이터 플랫폼으로,  
  GitHub에는 “**The Metadata Platform for your Data and AI Stack**” 및 “modern data stack을 위한 데이터 카탈로그”로 정의되어 있다.   
- 공식 사이트에서도 “leading open-source data catalog”이자 메타데이터 플랫폼으로, 데이터 디스커버리·거버넌스·데이터 인텔리전스를 제공한다고 명시한다.
- 다양한 DB, DWH, BI, 스트리밍, 파이프라인, ML 도구에 대해 폭넓은 통합 리스트를 제공하며, push/pull 기반 메타데이터 수집을 모두 지원한다. 
---

## 3. 아키텍처 및 설계 철학

### 3.1 OpenMetadata 아키텍처

- **중앙 메타데이터 저장소 + 통합 메타데이터 그래프** 구조:
  - 중앙 메타데이터 저장소를 기반으로, 데이터 자산·사용자·용어집·태그·품질 규칙·라인리지 등을 단일 그래프로 관리한다.
- **Ingestion Framework**:
  - 다양한 데이터 소스(데이터베이스, 데이터 웨어하우스, 대시보드, 파이프라인, ML 모델, 메시징, 스토리지, 기타 메타데이터 서비스)에서 메타데이터를 가져오기 위한 ingestion 프레임워크를 제공한다. 
- **확장 가능한 커넥터 모델**:
  - 커넥터 목록과 함께, ingestion 프레임워크 위에 커스텀 커넥터를 개발할 수 있는 가이드·예제가 제공된다. 
> OpenMetadata는 “하나의 통합 메타데이터 그래프 위에 디스커버리·품질·거버넌스·옵저버빌리티를 얹는 통합 플랫폼”이라는 색채가 강하다.

### 3.2 DataHub 아키텍처

- **분산형, 이벤트 기반 메타데이터 아키텍처**:
  - DataHub는 Kafka 기반 이벤트 스트림에 메타데이터 변경 로그를 기록하고, 이를 MySQL, Elasticsearch, (옵션) Neo4j 등 **세 가지 계층**으로 저장하는 분산 아키텍처를 사용한다. 
  - MySQL: 영속 메타데이터 저장  
  - Elasticsearch(OpenSearch): 검색 인덱스  
  - Neo4j(선택): 메타데이터 그래프 표현
- **Active Metadata / Actions**:
  - DataHub는 단순 조회형 카탈로그를 넘어서, 메타데이터 이벤트를 외부 시스템(Jira 등)과 연계하는 **Actions 프레임워크**를 제공하여 워크플로 자동화에 활용할 수 있다. 

> DataHub는 “메타데이터 이벤트 스트림을 중심으로 한 active metadata platform”이라는 포지셔닝이 강하며, 특히 대규모 분산 환경에서 메타데이터 변경을 실시간으로 반영하는 데 최적화되어 있다.

---

## 4. 핵심 기능 비교

### 4.1 공통 기능

두 프로젝트는 공통적으로 다음과 같은 기능을 제공한다.

- 데이터 카탈로그(데이터셋, 테이블, 컬럼 메타데이터 관리)  
- 데이터 디스커버리(검색·필터링, 태그, 용어집, 도메인) 
- 데이터 리니지(데이터 파이프라인 상의 상·하위 관계, 컬럼 레벨 리니지) 
- 데이터 거버넌스(용어집/글로서리, 태깅, 도메인, 오너십, 접근 권한 모델) 
- DataOps/DevOps 통합(REST·GraphQL API, CLI, CI/CD 연동, 알림·웹훅 등) 

### 4.2 OpenMetadata 강점 영역

- **데이터 품질·옵저버빌리티**:
  - OpenMetadata는 품질 규칙 정의, 데이터 프로파일링, 품질 리포트, SLA, 품질 알림 등 **데이터 품질·옵저버빌리티 기능**을 핵심 기능으로 강조한다.
- **통합 거버넌스 경험**:
  - 용어집, 분류체계, 데이터 제품, 도메인, 오너십, 승인 워크플로까지 한 UI에서 관리하는 **all-in-one 거버넌스 경험**에 초점을 둔다. 

> “품질·옵저버빌리티·거버넌스까지 한 번에 가져가고 싶다”는 요구가 강할수록 OpenMetadata 쪽 설계가 더 직관적으로 느껴질 수 있다.

### 4.3 DataHub 강점 영역

- **AI & Data Context Management**:
  - DataHub는 “AI & Data Context Management”라는 메시지와 함께, AI/LLM이 안전하게 데이터를 사용할 수 있도록 컨텍스트를 제공하는 플랫폼으로 포지셔닝한다.
- **Active Metadata & Actions**:
  - 메타데이터 이벤트를 트리거로 Jira 티켓 생성, 알림, 승인 워크플로 등 **메타데이터 기반 자동화**를 구성할 수 있는 Actions 프레임워크를 제공한다. 

> “대규모 조직에서 메타데이터를 중심으로 워크플로/자동화/AI를 엮고 싶다”는 요구가 강하면 DataHub 쪽이 더 어울릴 가능성이 크다. 

---

## 5. 기술 스택 및 구성 요소

### OpenMetadata

- 중앙 메타데이터 저장소(관계형 DB) + 검색 인덱스(예: Elasticsearch) 기반으로 동작하며, Helm chart에서는 MySQL·Elasticsearch 조합을 기본 배포로 사용한다.
- Ingestion framework는 Python 기반의 워크플로로 다양한 소스에 연결하여 메타데이터를 수집한다. 

### DataHub

- DataHub는 MySQL, Elasticsearch/OpenSearch, (옵션) Neo4j, Kafka를 핵심 구성 요소로 사용하는 분산형 아키텍처를 채택한다. 
- ingestion 서버 및 GMS(Graph Metadata Service)는 Docker/Helm 기반으로 배포되며, Kafka를 통한 메타데이터 commit log를 사용한다.

> 두 프로젝트 모두 웹 UI + API 레이어 + 메타데이터 저장소 + 검색 인덱스 + ingestion 레이어로 구성된 “전형적인 메타데이터 플랫폼 아키텍처” 구조를 가진다.  

---

## 6. 커넥터·통합(Integrations) 비교

### 6.1 DB / DWH / Data Lake

**OpenMetadata**

- 공식 Connectors 문서에 DB/DWH/스토리지 커넥터가 정리되어 있으며,  
  MySQL, PostgreSQL, MSSQL, Oracle, Snowflake, BigQuery, Redshift, Databricks, Vertica, Teradata 등 다양한 시스템을 지원한다.  
- 스토리지·레이크 계층에서는 S3/GCS/ADLS 기반 Datalake 및 Storage 커넥터를 제공한다.  

**DataHub**

- Integrations 페이지에서 MySQL, PostgreSQL, Oracle, Snowflake, BigQuery, Redshift, Vertica, Teradata, Databricks 등 폭넓은 DB/DWH를 지원하며,  
  S3 Data Lake, Azure Blob Storage, Dremio, CockroachDB 등도 목록에 포함한다. 

> 대표적인 클라우드 DWH(Snowflake, BigQuery, Redshift, Databricks)와 RDBMS(MySQL, Postgres, MSSQL, Oracle)는 양쪽 모두 잘 지원하므로, 이 범주만으로는 우열을 가리기 어렵다. 

### 6.2 BI / Dashboard

**공통 지원 (대표적)**

- Tableau, Power BI, Looker, Apache Superset, Metabase, Redash, Qlik 등 주요 BI·대시보드 도구에 대해 양쪽 모두 커넥터를 제공한다.

> 일반적인 BI 스택(Tableau/Power BI/Looker/Superset/Metabase 등)을 쓰는 조직이라면, BI 커넥터 측면에서 두 프로젝트 모두 실무 요구를 충족한다고 보는 것이 합리적이다. 

### 6.3 파이프라인·오케스트레이션

**OpenMetadata**

- Airflow, Dagster, dbt, Glue, Fivetran, Flink, NiFi, Kafka Connect, Databricks Pipeline 등 다양한 파이프라인·오케스트레이션 도구 커넥터를 제공한다.   

**DataHub**

- Airflow, Dagster, Prefect, dbt, Glue, Fivetran, NiFi 등 현대적인 ETL/ELT·오케스트레이션 도구에 대한 ingestion를 지원하며, push/pull 방식 모두 가능하다.   

### 6.4 스트리밍 / 메시징

**OpenMetadata**

- Kafka, Pulsar, Kinesis, Redpanda 등 메시징·스트리밍 플랫폼에 대한 커넥터를 제공한다.

**DataHub**

- Kafka, Kafka Connect, Pulsar 등에 대한 통합을 제공한다.

### 6.5 ML·Feature Store·기타

**OpenMetadata**

- ML Model Services로 MLflow, SageMaker(및 최근 버전에서 Vertex AI까지)를 지원하며, ML 모델 메타데이터를 플랫폼 내에서 관리할 수 있다. 

**DataHub**

- ML/Feature Store/AI 플랫폼과의 통합으로 MLflow, Feast(Feature Store), SageMaker, Vertex AI 등을 지원하며, AI·ML 워크플로와 메타데이터를 연계할 수 있다.  

> ML·Feature Store·Vertex AI 등까지 쓰는 조직에서는 DataHub가 약간 더 폭넓은 통합 옵션을 제공하는 편이다. 

---

## 7. 배포·운영 관점 비교

### OpenMetadata

- **Helm Charts**:
  - `openmetadata-helm-charts` 저장소를 통해 Kubernetes용 Helm 차트를 제공하며, OpenMetadata와 MySQL, Elasticsearch를 함께 배포할 수 있다. 
- **클라우드 인프라 템플릿**:
  - Terraform 모듈을 통해 AWS 상에 OpenMetadata 환경을 구성하는 예제가 제공된다.

### DataHub

- **Helm Charts**:
  - 공식 문서에서 Kubernetes 상에 DataHub와 의존성(Elasticsearch, MySQL, Kafka, 옵션 Neo4j)을 배포하기 위한 Helm 차트를 제공한다. 
- **Ingestion Docker**:
  - 별도의 ingestion Docker 이미지를 제공해, CI/CD 파이프라인이나 배치 환경에서 메타데이터 ingestion 작업을 쉽게 실행할 수 있다.

> 둘 다 Docker·Helm 기반 배포를 공식 지원하므로, 운영 난이도는 “조직이 이미 MySQL/Kafka/Elastic 계열 인프라에 익숙한 정도”에 더 영향을 받을 가능성이 크다.  

---

## 8. 라이선스·커뮤니티·성숙도

### 라이선스

- **OpenMetadata**: Apache License 2.0  
- **DataHub**: Apache License 2.0 

### 커뮤니티 및 생태계

- OpenMetadata
  - 공식 사이트에서 3,000+ 엔터프라이즈 배포, 11,000+ OSS 멤버, 370+ 코드 컨트리뷰터를 언급하며 빠르게 성장하는 오픈소스 메타데이터 플랫폼이라고 밝힌다.   
- DataHub
  - LinkedIn 엔지니어링 블로그, 여러 컨퍼런스 발표, 다양한 기업 도입 사례, 2025 로드맵 등 풍부한 레퍼런스를 보유하고 있으며, “#1 open-source data catalog”로 언급된다. 

> 역사·사례 측면에서 DataHub가 약간 더 오래되고 레퍼런스가 풍부하며, 최근 몇 년 간 성장 속도만 보면 OpenMetadata가 매우 가파른 성장세를 보이는 것으로 이해하는 것이 일반적이다. 이것은 추론입니다  

---

## 9. 선택 가이드 (어떤 상황에서 무엇을 쓸까?)

아래는 위 정보에 기반한 **의사결정용 가이드**로, 문장 전체가 해석·판단에 해당한다. **이것은 추론입니다**

1. **“데이터 품질·거버넌스·옵저버빌리티까지 한 번에”가 목표일 때**
   - 품질 규칙, 프로파일링, SLA, 거버넌스 워크플로를 하나의 UI와 모델로 관리하고 싶다면  
     → **OpenMetadata**를 먼저 PoC 후보로 고려

2. **“대규모 분산 환경 + 메타데이터 이벤트 기반 자동화/AI 연계”가 중요할 때**
   - Kafka 중심의 메타데이터 이벤트 스트림, Jira·Slack 등과 연동되는 Actions, AI/LLM 컨텍스트 관리가 핵심이면  
     → **DataHub** 쪽이 자연스러운 선택

3. **스택 조합으로 보는 관점**
   - `Snowflake(or BigQuery/Redshift/Databricks) + dbt + Airflow + Tableau/PowerBI` 같은 “전형적인 현대 데이터 스택”에서는  
     - 두 플랫폼 모두 커버 가능 (커넥터 측면에서 큰 차이 없음)  
     - 거버넌스/품질을 강하게 밀고 싶으면 OpenMetadata,  
       active metadata/AI·워크플로에 더 무게를 두면 DataHub 쪽에 우선순위를 두는 방식 추천
   - `Feast + Vertex AI + Hudi + Dremio + Kafka` 등의 조합처럼  
     - Feature Store·Vertex AI·Hudi·Dremio 같은 컴포넌트 비중이 크면 DataHub가 약간 더 유리

4. **조직의 운영 역량·기존 인프라**
   - 이미 Kafka·MySQL·Elasticsearch/OpenSearch 환경을 널리 쓰고 있고, 이벤트 중심 아키텍처에 익숙한 팀이라면
     → DataHub 도입이 상대적으로 자연스러움  
   - 단일 중앙 메타데이터 DB + ingestion 워크플로 형태를 선호하고, 품질/거버넌스 팀과의 협업이 중요하다면
     → OpenMetadata가 더 직관적인 모델로 느껴질 수 있음

---

## 10. 결론 요약

- **OpenMetadata**는  
  - “통합 메타데이터 그래프 위에서 데이터 디스커버리, 품질, 거버넌스, 옵저버빌리티를 한 번에 제공하는 통합 플랫폼”에 가깝고,  
  - 데이터 품질·거버넌스·협업 경험에 강점을 둔다.  

- **DataHub**는  
  - “현대 데이터·AI 스택을 위한 메타데이터 허브이자 데이터 카탈로그”로,  
  - Kafka 기반 분산 아키텍처, active metadata, Actions, AI/LLM 컨텍스트 관리에 초점을 둔다. 

> 실제 선택은 “현재/미래에 사용할 데이터·AI 스택 구성, 거버넌스 성숙도, 이벤트 기반 아키텍처 선호 여부, 팀의 운영 역량”에 따라 달라질 것이며,  
> 소규모 PoC 환경에서 두 제품을 동일한 소스(예: Snowflake + dbt + Airflow)로 비교해 보는 것이 가장 확실한 검증 방법이다. 이것은 

## 11. 참고 링크
[OpenMetadata/DataHub](https://www.linkedin.com/pulse/open-metadata-vs-datahub-choosing-right-data-catalog-tool-salih-j8xff)

[Whichoneisbetter](https://atlan.com/openmetadata-vs-datahub/)

[Reddit토론](https://www.reddit.com/r/dataengineering/comments/yxrh9y/metadata_store_which_one_to_choose_openmetadata/)