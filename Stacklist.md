# 0. 기술 스택 목록
**강조** <-Datahub 공식 커넥터 존재
- Data Governance / Catalog:  
  **DataHub OSS**

- Data Quality:  
  **Great Expectations**, *Soda Core*, *Deequ*

- Monitoring & Logging:  
  *Prometheus*, **Grafana**, *Loki*, **OpenSearch**, *Jaeger*, *OpenTelemetry* *Debezium*,

- Ingestion:  
  *Airbyte OSS*, *Singer*, *Meltano*, **Apache NiFi**,  *Debezium*, *Fluent Bit*, *Fluentd*

- Streaming:  
  **Apache Kafka**, **Apache Pulsar**, *Apache Flink*,  *Apache Storm*

- Storage / Lakehouse:  
  Object Storage (**S3**, **GCS**, , *MinIO*),  
  *Apache Iceberg*, **Delta Lake**, *Apache Hudi*, **Apache Hive**,  
  *Apache Parquet*, *Apache ORC*

- Query Engine:  
  **Trino**, **Apache Spark** ,**OpenTelemetry**, **PrestoDB**, **Apache Hive**, *Apache Impala*,

- Modeling:  
  **dbt Core**, , *Apache Beam*, 

- Orchestration:  
  **Apache Airflow**, **Dagster**, **Prefect OSS**






---
# 1. Ingestion ↔ Storage 호환성

## 1.1 Airbyte / Singer / Meltano / NiFi ↔ Object Storage / Lakehouse

### Airbyte

- 데스티네이션으로 S3, GCS, Azure Blob, MinIO 등 객체 스토리지를 지원.  
- S3 Data Lake 커넥터는 Iceberg 포맷을 직접 지원해  
  `Airbyte → S3(Data Lake) → Iceberg` 구성이 가능하다.  

### Singer / Meltano

- Singer는 `tap`(소스) / `target`(목적지) 구조로, target을 S3/DB/DW 등으로 설정.  
- 파일 포맷은 주로 Parquet/JSON이므로 Iceberg/Delta/Hudi 인제스트 전단에 두기 좋다.

### Apache NiFi

- 다양한 프로세서로 S3/HDFS/Kafka/DB 등으로 쓰기 지원.  
- 레이크하우스 앞단에서 파일/로그/IoT 인제스트용으로 적합하다.

### Fluent Bit / Fluentd

- 로그/메트릭 수집기로, S3/Elasticsearch/OpenSearch/Kafka 등 다양한 출력 지원.  
- “로그 데이터 레이크”나 Kafka 전단 수집에 자연스럽게 사용 가능.

## 1.2 Kafka Connect / Debezium ↔ Streaming / Storage

### Kafka Connect

- 수많은 소스/싱크 커넥터로 DB↔Kafka, Kafka↔S3/HDFS/ES 등을 연결하는 스트리밍 통합 프레임워크.  

### Debezium

- MySQL/Postgres 등 DB 트랜잭션 로그를 CDC 이벤트로 Kafka에 발행하는 프레임워크.  

> 대표 흐름 예시:  
> `RDB → Debezium(Kafka Connect) → Kafka → Spark/Flink → Iceberg/Delta/Hudi`  
> 배치·실시간 모두 같은 레이크하우스 포맷으로 수렴시키는 구조를 만들 수 있다.  
# 2. Storage / Lakehouse를 중심으로 본 호환성

## 2.1 Iceberg / Delta / Hudi ↔ 엔진들

### Apache Iceberg

- **Trino**  
  - Iceberg 전용 커넥터 제공, Iceberg Table Spec v1/v2 지원.  
- **Spark**  
  - Spark SQL/DataFrame에서 Iceberg 테이블 생성·조회 공식 가이드 및 예제 존재.  
- **Flink**  
  - Iceberg는 Spark/Trino/Flink 등 다중 엔진 환경을 목표로 설계되었고, Flink 커넥터도 제공.  

### Delta Lake

- Spark, PrestoDB, Flink, Trino, Hive 등 여러 엔진에서 사용 가능.  
- **Delta UniForm** 옵션을 통해 Iceberg/Hudi 클라이언트로도 Delta 테이블을 읽을 수 있어 포맷 간 브릿지 역할을 수행.  

### Apache Hudi

- Uber에서 시작된 레이크하우스 테이블 포맷으로, Spark·Flink 기반 ingest/처리와 Presto/Trino 쿼리 통합을 제공.  

> 결론:  
> Iceberg / Delta / Hudi 모두 Spark·Trino·Flink와 함께 쓰는 패턴이 널리 쓰이며,  
> “S3 + Iceberg + Spark + Trino” 같은 조합은 대표적인 오픈소스 레이크하우스 패턴이다.  


---


# 3. Streaming ↔ Engine / Transform 호환성

## 3.1 Kafka / Pulsar ↔ Spark / Flink / Kafka Streams

### Apache Kafka

- Spark Structured Streaming에서 기본 소스로 지원되며,  
  Flink·Kafka Streams·Storm 등 대부분 스트리밍 엔진에서 사용중

### Apache Pulsar

- Flink, Spark 및 Kafka API 호환 레이어와의 통합을 제공하는 플러그인/커넥터들이 있다.  

### Spark ↔ Kafka

- Spark Structured Streaming으로 Kafka 토픽을 읽어 Iceberg/Delta/Hudi에 쓰는 패턴이 널리 사용된다.  

### Flink ↔ Kafka / Delta / Hudi

- Flink는 Kafka 소스를 기본 지원하고, Delta·Hudi에서도 Flink 통합 및 ingest 기능을 문서화하고 있다.  

> 스트리밍 레이어 관점에서  
> `Kafka(or Pulsar) + Spark/Flink` 조합은 호환성과 사례가 모두 충분한 “안전한 선택”이다.  


---

# 4. Query Engine / Modeling ↔ Lakehouse 호환성

## 4.1 Trino / Spark / Presto ↔ Iceberg / Delta / Hudi

### Trino ↔ Iceberg

- Trino는 Iceberg 전용 커넥터로 Iceberg Table Spec v1/v2를 지원한다.  

### Spark ↔ Iceberg

- Spark SQL·DataFrame에서 Iceberg 테이블 생성·조회가 가능하며, 모두 예제가 풍부하다.  

### Delta Lake ↔ Spark / Flink / Trino

- Delta Lake는 Spark, Presto/Trino, Flink, Hive 등 다양한 엔진을 지원하며, Delta Kernel/커넥터로 비-Spark 환경도 커버한다.  

### Hudi ↔ Spark / Presto / Trino

- Hudi는 Spark/Presto/Trino와의 통합을 전제로 한 레이크하우스 테이블 포맷으로, ingest와 스트리밍 처리에 강점이 있다.  

### DuckDB / ClickHouse / Impala

- 이 엔진들은 직접 Iceberg/Delta/Hudi를 완전 지원하는 경우는 제한적이지만,  
  `Parquet on S3` 또는 Hive Metastore를 통해 간접적으로 레이크하우스 테이블을 읽는 패턴이 가능하다.  

## 4.2 dbt Core ↔ Trino / Spark

### dbt-trino 어댑터

- dbt 공식 생태계 및 Trino 커뮤니티에서 제공하는 어댑터로,  
  `pip install dbt-core dbt-trino` 형태로 설치하며 Trino를 백엔드로 Iceberg/Delta/Hudi 위에서 모델링할 수 있다.  

### dbt-spark 어댑터

- Spark SQL을 통해 Delta/Hive 등을 변환하는 패턴이 일반적이며, Airflow·Spark·dbt·Trino·Hive Metastore를 묶은 PoC 예제도 다수 존재한다.  

> 요약:  
> `Iceberg/Delta/Hudi + Trino/Spark + dbt` 조합은  
> 레이크하우스 + SQL 변환 관점에서 호환성과 실사례가 풍부한 축이다.  


---

# 5. Orchestration /  / Quality 호환성 (DataHub 기준)

## 5.1 Airflow / Dagster / Prefect ↔ 나머지 스택

### Apache Airflow

- Spark, Trino, dbt, Airbyte, Kafka 등 거의 모든 컴포넌트를 Operator/Bash/Shell/Sensor로 제어 가능하다.  
- `acryl-datahub-airflow-plugin`을 통해 DAG/Task/Run/컬럼 계보를 DataHub로 전송하는 공식 플러그인이 있다.  

### Dagster / Prefect

- Python 기반 오케스트레이터로 Spark, dbt, Kafka, Airbyte 등을 코드 레벨에서 쉽게 호출할 수 있다.  
- DataHub에는 Airflow만큼의 공식 플러그인은 없지만,  
  OpenLineage 또는 DataHub SDK(REST/GraphQL)를 이용해 `DataJob`/`Dataset` 계보를 간접 푸시하는 패턴이 가능하다.  


## 5.2 Data Quality / Monitoring

### Great Expectations / Soda / Deequ

- **Great Expectations**는 DataHub와 공식 통합을 제공하며, Validation 결과를 메타데이터로 push하는 액션 클래스를 제공한다.  
- **Soda / Deequ**는 테스트 결과를 테이블/로그로 남기고, 이를 DataHub·OpenSearch·Grafana에서 조회하는 간접 패턴이 일반적이다.  

### Prometheus / Grafana / Loki / OpenSearch

- **Prometheus**는 각 컴포넌트 메트릭 수집에 사용.  
- **Grafana**는 DataHub와 통합되어 대시보드 메타데이터를 인제스트할 수 있다.  
- **Loki / OpenSearch**는 로그/검색 백엔드로 사용되며, DataHub는 OpenSearch/Elasticsearch를 내부 검색·그래프 백엔드로 공식 활용한다.  


---

# 6. 정리: 상호 호환성이 높은 기본 조합 예시

소규모 팀 + 오픈소스 + DataHub 기준, 호환성과 사례가 풍부한 “안전한 기본 스택” 예시는 다음과 같다.
## 6.0 Governance / Quality / Monitoring

- **DataHub OSS**  
- **Great Expectations**  
- **Prometheus / Grafana / Loki / OpenSearch**

## 6.1 Ingestion

- **Airbyte**: Batch/SaaS 수집 → S3/MinIO(+ Iceberg) 적재  
- **Debezium + Kafka Connect**: RDB CDC → Kafka 스트림

## 6.2 Streaming

- **Apache Kafka** (필요 시 Pulsar 대체 가능)

## 6.3 Storage / Lakehouse

- **Object Storage (S3/MinIO) + Apache Iceberg**  
- (또는 **Delta Lake**, 필요 시 UniForm로 Iceberg/Hudi 클라이언트와 겸용)  

## 6.4 Query / Transform

- **Trino + dbt Core (`dbt-trino`)**  
- **Spark** (배치/스트리밍 ETL, 필요 시 ML까지)

## 6.5 Orchestration

- **Apache Airflow**


이 조합은  
- 레이크하우스 관점에서 포맷/엔진 간 호환성이 높고,  
- DataHub·Airflow·dbt 기준으로 계보·거버넌스를 일관되게 가져갈 수 있어,  
팀이 “오픈소스 중심 데이터 파이프라인”을 설계하기에 안정적인 선택지로 볼 수 있다.  
