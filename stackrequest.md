- [1. 목적](#1-목적)
- [2. 구축 목표 및 범위](#2-구축-목표-및-범위)
- [3. 기술 스택 구성 요소](#3-기술-스택-구성-요소)
  - [3.1 거버넌스 / 품질 / 모니터링](#31-거버넌스--품질--모니터링)
  - [3.2 오케스트레이션](#32-오케스트레이션)
  - [3.3 데이터 수집 (Ingestion)](#33-데이터-수집-ingestion)
  - [3.4 저장 및 테이블 포맷](#34-저장-및-테이블-포맷)
  - [3.5 처리 엔진 및 변환](#35-처리-엔진-및-변환)
  - [3.6 데이터 소비 / BI](#36-데이터-소비--bi)
- [4. 기술 스택 간 호환성 및 권장 설치 구조](#4-기술-스택-간-호환성-및-권장-설치-구조)
  - [4.1 스트림 계층 (Kafka 중심)](#41-스트림-계층-kafka-중심)
  - [4.2 레이크하우스 계층 (S3 + Iceberg)](#42-레이크하우스-계층-s3--iceberg)
  - [4.3 오케스트레이션 및 쿼리-게이트웨이](#43-오케스트레이션-및-쿼리-게이트웨이)
  - [4.4 거버넌스 / 품질 계층 연계](#44-거버넌스--품질-계층-연계)
- [5. 버전 및 런타임 관점의 기본 전제](#5-버전-및-런타임-관점의-기본-전제)
- [6. 인프라팀에 요청드리는 사항](#6-인프라팀에-요청드리는-사항)



# 1. 목적

본 요구서는 당사 데이터 플랫폼 구축을 위해 필요한 애플리케이션 레벨 기술 스택 및 호환성 기반 설치 방향을 인프라팀에 공유하기 위한 문서입니다.  
서버 사양, 네트워크, 보안, 스토리지 용량 등 인프라 상세 설계는 인프라팀에서 별도 검토하되,  
아래 기술 스택 간의 호환성과 표준적인 아키텍처 패턴을 고려한 설치 방향을 제안합니다.

---

# 2. 구축 목표 및 범위

1. 배치·실시간(스트리밍)·CDC를 포함하는 통합 데이터 파이프라인 구축  
2. S3 + Apache Iceberg 기반 레이크하우스(Lakehouse) 환경 구성  
3. Spark/Trino 중심의 분석·모델링 및 BI 환경 제공  
4. DataHub, Great Expectations를 활용한 메타데이터 관리 및 품질·모니터링 체계 구축  

본 요구서는 위 목표를 달성하기 위한 애플리케이션/플랫폼 계층의 구성요소와 이들 간의 호환성·연계 구조를 정의합니다.

---

# 3. 기술 스택 구성 요소

## 3.1 거버넌스 / 품질 / 모니터링

- 메타데이터 카탈로그: DataHub  
- 데이터 품질 검증: Great Expectations  
- 로그/메트릭 모니터링: Grafana, OpenSearch(엘라스틱서치 계열)  

## 3.2 오케스트레이션

- 배치 워크플로우: Apache Airflow  
- 스트림 처리 오케스트레이션/엔진: Apache Flink  

## 3.3 데이터 수집 (Ingestion)

- 배치 수집: Airbyte (Batch)  
- CDC 배치 수집: Airbyte CDC  
- CDC 스트림 수집: Debezium (+ Kafka)  
- 스트리밍 수집: Apache Kafka  

## 3.4 저장 및 테이블 포맷

- 데이터 레이크: S3 호환 스토리지 (예: AWS S3, MinIO 등)  
- 테이블 포맷: Apache Iceberg  

## 3.5 처리 엔진 및 변환

- 분산 처리 엔진: Apache Spark  
- SQL 쿼리 엔진: Trino  
- SQL 기반 변환/모델 관리: dbt Core (dbt-spark, dbt-trino 어댑터)  

## 3.6 데이터 소비 / BI

- BI 및 대시보드: Apache Superset, Metabase  

---

# 4. 기술 스택 간 호환성 및 권장 설치 구조

## 4.1 스트림 계층 (Kafka 중심)

- Debezium → Kafka  
  - RDBMS 변경 데이터를 Kafka 토픽으로 CDC 이벤트 발행  
- Airbyte CDC → Kafka  
  - CDC 기반의 외부 시스템 변경 데이터 수집  
- Kafka ↔ Flink / Spark  
  - Flink: Kafka를 Source/Sink로 공식 지원  
  - Spark: Structured Streaming으로 Kafka 읽기/쓰기 지원  

**결론:** 스트림 계층은 Kafka를 중심으로 Debezium, Airbyte, Flink, Spark가 연계되도록 설치하는 것이 표준적이며 상호 호환성이 높습니다.

---

## 4.2 레이크하우스 계층 (S3 + Iceberg)

- S3: 모든 원천/가공 데이터를 저장하는 Object Storage 레이어  
- Apache Iceberg: S3 위에 구성되는 테이블 포맷  
- Spark / Trino / Flink / dbt:
  - Spark, Trino, Flink는 Iceberg 커넥터/카탈로그를 통해 동일 Iceberg 테이블을 읽기/쓰기 가능  
  - dbt는 dbt-spark, dbt-trino 어댑터를 통해 Iceberg 기반 테이블에 대한 SQL 변환/모델 관리 수행  

**결론:**  
데이터 저장은 S3, 테이블 포맷은 Iceberg로 일원화하고,  
Spark · Trino · Flink · dbt가 동일 Iceberg 테이블을 공유하는 구조로 설치하는 것이 가장 자연스럽고 호환성이 높습니다.

---

## 4.3 오케스트레이션 및 쿼리-게이트웨이

- Airflow  
  - Airbyte Job 호출, Debezium 관련 작업(스크립트), Spark Job, dbt CLI 실행 등을 하나의 DAG에서 오케스트레이션  
- Trino  
  - BI 도구(Superset, Metabase)에서 접속하는 단일 SQL/쿼리 게이트웨이 역할  
  - Trino를 통해 Iceberg, S3, 기존 RDBMS 등 다양한 소스를 통합 조회  

**결론:**  
- Airflow는 인제스트 → 처리 → 모델링 → 품질검증 → 적재를 연결하는 오케스트레이터로 설치  
- Trino는 BI 및 Ad-hoc Query를 위한 대표 쿼리 엔진으로 설치하는 구조가 호환성과 운영성 측면에서 적합합니다.

---

## 4.4 거버넌스 / 품질 계층 연계

- DataHub  
  - Airflow, Kafka, Spark, Trino, dbt, Iceberg, Superset/Metabase 등과 공식 커넥터를 통해 메타데이터/라인리지 수집 가능  
- Great Expectations  
  - Spark DataFrame, SQL 테이블, S3 파일(Parquet/CSV 등)을 대상으로 데이터 품질 검증 수행  
  - Airflow DAG 내에서 Validation Task로 호출 가능  

**결론:**  
현재 제안된 스택 전체와 DataHub/Great Expectations는 공식 커넥터/플러그인을 통해 호환되며,  
추가적인 대체 기술 없이 그대로 통합 가능합니다.

---

# 5. 버전 및 런타임 관점의 기본 전제

인프라 상세 설계와는 별개로, 아래와 같은 공통 전제를 요청드립니다.

1. JVM(Java) 버전 통일  
   - Kafka, Debezium, Flink, Spark, Trino, Iceberg 등이 모두 JVM 기반이므로  
     Java 11 또는 17 등 단일 표준 버전으로의 운영을 전제로 합니다.

2. Python 버전 통일  
   - Airflow, dbt, Great Expectations, 일부 DataHub 인젝터 등이 Python 기반이므로  
     Python 3.10 또는 3.11 등 하나의 Python 버전으로 통일하여 운영하는 것을 요청드립니다.

3. Iceberg – 엔진 버전 매트릭스 검토  
   - 선택된 Iceberg 버전이 Spark, Trino, Flink에서 공식 지원되는 조합인지  
     인프라/플랫폼 레벨에서 버전 매트릭스 검토를 요청드립니다.

4. Kafka – Debezium 버전 호환성 검토  
   - Kafka 브로커 및 Debezium 커넥터 간 공식 호환 버전 조합 사용을 전제로 합니다.

---

# 6. 인프라팀에 요청드리는 사항

1. 본 요구서에 명시된 기술 스택 간 호환성 및 추천 설치 구조를 기반으로,  
   - 서버(노드 수, CPU/RAM, 디스크 등)  
   - 네트워크(VPC/Subnet, 보안 그룹, 포트 설계)  
   - 스토리지(S3/오브젝트 스토리지, 메타스토어 DB 등)  
   - 보안/인증(내부 접속 정책, SSO 연동 여부, TLS 등)  
   에 대한 인프라 설계 초안 검토를 요청드립니다.

2. 필요 시,  
   - 컴포넌트별 예상 부하(개략적인 데이터 규모, 처리 주기)  
   - 환경 분리 전략(Dev / Staging / Prod)  
   - 운영/모니터링 요구사항(로그/메트릭 수집 범위)  
   등을 추가로 정리하여 제공할 수 있습니다.
