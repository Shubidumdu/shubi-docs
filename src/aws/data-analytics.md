# Data & Analytics

## Amazon Athena

- S3에 저장된 데이터를 분석하기 위한 **서버리스** 쿼리 서비스
- 파일을 쿼리하기 위해 표준 SQL 언어를 사용 (Presto)
- CSV, JSON, ORC, Avro, Parquest 지원
- 비용: 스캔된 데이터 1TB당 $5.00
- 일반적으로 Amazon Quicksight와 함께 사용하여 리포트/대쉬보드 생성
- **사례**: 비즈니스 인텔리전스 / 분석 / VPC Flow Logs, ELB Logs, **CloudTrail trails** 등에 대한 리포팅 & 분석 & 쿼리
- **팁**: 서버리스 SQL로 S3 내 데이터를 분석하고 싶다? -> Athena를 쓴다!

### Athena - Performance Improvement

- 비용 절감을 위해서는 **columnar data**를 사용할 것 (스캔이 덜 일어남)
  - Apache Parquet 또는 ORC 추천
  - 높은 성능 향상
  - Parquet 또는 ORC로 데이터를 변환하려면 Glue를 사용
- 더 작은 검색을 위해 **데이터 압축** (bzip2, gzip, lz4, snappy, zlip, zstd...)
- 가상 컬럼을 쉽게 쿼리하기 위해 S3 내 데이터셋을 **파티션**
  - s3://yourBucket/pathToTable
    - /\<PARTITION_COLUMN_NAME\>=\<VALUE\>
      - /\<PARTITION_COLUMN_NAME\>=\<VALUE\>
        - ...
  - ex. `s3://athena-examples/flight/parquet/year=1991/month=1/day=1/`
- 오버헤드를 줄이기 위해서는 **큰 파일**(> 128MB)을 사용

### Athena - Federated Query

- 관계형/비관계형/오브젝트/커스텀 데이터 소스(AWS든 온-프레미스든)에 저장된 데이터 전반에 SQL 쿼리를 실행할 수 있음
- Data Source Connector를 사용하여 AWS Lambda가 Federated Queries를 실행하도록 할 수 있음 (e.g., CloudWatch logs, DynamoDB, RDS, ...)
- 그 결과도 S3에 다시 저장

![Federated Query](https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2019/11/27/AthenaQueryPic1.png)

## Redshift

- Redshift는 PostgreSQL 기반이지만, **OLTP(online transaction processing)를 위해 사용되는 것이 아님**
- **OLAP**임 - **online analytical processing (분석 및 데이터 웨어하우싱)
- 다른 데이터 웨어하우스보다 10배의 성능 & PB(페타바이트) 단위로 확장
- **Columnar** storage of data (row 기반 대신) & 병렬 쿼리 엔진
- 프로비전된 인스턴스에 기반하여 비용 지불
- 쿼리 수행에 SQL 인터페이스 사용 가능
- Amazon Quicksight 또는 Tableau와 같은 BI(Business intelligence) 툴과 호환
- **vs Athena**: 인덱싱 덕분에 쿼리 / 조인 / 병합(aggregation)이 더 빠름

### Redshift - Cluster

![Redshift Cluster Example](https://editor.analyticsvidhya.com/uploads/298992.png)

- Leader node: 쿼리 플래닝, 결과 병합 목적
- Compute node: 쿼리 수행, leader node에게 결과 전달
- 미리 노드 사이즈를 프로비전해야함
- 비용 절감을 위해 Reserved Instance 사용할 수 있음

### Redshift - Snapshots & DR

- **Redshift는 대부분의 클러스터에 단일 AZ이지만, 일부 클러스터에 Multi-AZ 모드를 보유함**
- 스냅샷은 클러스터 하나의 point-in-time 백업이며, 내부적으로 S3에 저장됨
- 스냅샷은 incremental함 (변경이 이루어진 것만 저장됨)
- **새 클러스터**로 스냅샷을 복구할 수 있음
- Automated: 매 8시간, 매 5GB, 또는 스케줄에 따라, retention을 설정
- Manual: 스냅샷을 직접 삭제하기 전까지 보존됨
- **Amazon Redshift가 자동으로 클러스터의 스냅샷(automated or manual)을 다른 AWS 리전으로 복사하도록 설정할 수 있음 (disaster recovery)**

### Redshift - Loading data into Redshift

- **insert 작업은 거대하게 처리하는 편이 훨씬 좋음**
- 다음을 통해 데이터를 가져올 수 있음
  - Amazon Kinesis Data Firehose - S3 copy를 통해
  - S3 Bucket - 인터넷 또는 VPC를 통해서
  - EC2 Instance (/w JDBC driver) ~ 배치 단위로 데이터 write를 하는게 더 좋음

### Redshift - Spectrum

- 이미 S3에 있는 데이터를 따로 가져오지 않고도 쿼리
- **쿼리를 시작하기 위해 사용가능한 Redshift 클러스터를 보유해야함**
- 쿼리는 수천개의 Redshift Spectrum 노드에 전달됨
