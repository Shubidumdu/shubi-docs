# Disaster Recovery & Migrations

- 기업의 사업 지속성이나 재정에 부정적인 영향을 끼치는 어떤 상황을 **disaster**라고 함
- Disaster recovery (DR)은 disaster에 대비 및 복구를 하는 것을 의미
- Disaster recovery의 종류?
  - 온-프레미스 => 온-프레미스: 전통적인 방식의 DR, 매우 비쌈
  - 온-프레미스 => AWS 클라우드: 하이브리드 방식
  - AWS Cloud Region A => AWS Cloud Region B
- 복구 목표에 대한 두 용어:
  - RPO: Recovery Point Objective
  - RTO: Recovery Time Objective

## RPO and RTO

![RPO & RTO](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/31/ClouldOps_913_1.png)

## Disaster Recovery Strategies

- Backup & Restore
- Pilot Light
- Warm StandBy
- Hot Site / Multi Site Approach

![Disaster Recovery Strategies](https://d2908q01vomqb2.cloudfront.net/fc074d501302eb2b93e2554793fcaf50b3bf7291/2021/04/02/Figure-2.png)

### Backup and Restore (High RPO)

![Backup & Restore](https://velog.velcdn.com/images/chan9708/post/e713d1f4-6b80-4417-96ee-9869ad661cb3/image.png)

### Pilot Light

- 앱의 작은 버전이 항상 클라우드에서 실행되도록 함
- 핵심 코어에 유용함 (pilot light)
- Backup and Restore와 매우 유사
- Backup and Restore보다 빠름 ~ 주요 critical system이 항상 가동 중이기 때문

![Pilot Light](https://static.packt-cdn.com/products/9781838645649/graphics/assets/9bb1d8f6-28c4-4ae0-a198-2709e2a33cc7.png)

### Warm Standby

- 전체 시스템을 가동 및 실행시키지만, 최소한의 사이즈로 유지함
- disaster 발생 시에는 프로덕션의 부하에 맞추어 스케일 업을 수행할 수 있음

![Warm Standby](https://velog.velcdn.com/images/chan9708/post/1dcdc96b-8324-4cb5-9f50-a1cd8d3fdaca/image.png)

### Multi Site / Hot Site Approach

- 매우 낮은 RTO (몇분 또는 몇초 단위) - 매우 비쌈
- Full Production Scale이 AWS와 온-프레미스로 실행됨

![Multi Site / Hot Site Approach](https://velog.velcdn.com/images/chan9708/post/7e3f87f2-5389-47cd-967d-6097eb16a813/image.png)

## Disaster Recovery Tips

- **Backup**
  - EBS 스냅샷, RDS 자동화 백업 / 스냅샷 등..
  - 정기적인 푸쉬 ~ S3 / S3 IA / Glacier, Lifecycle Policy, Cross Region Replication
  - 온-프레미스의 경우: Snowball 똔느 Storage Gateway

- **High Availability**
  - 리전에서 리전으로 DNS를 마이그레이션하는 경우 Route53을 사용
  - RDS Multi-AZ, ElastiCache Multi-AZ, EFS, S3
  - Direct Connect에서는 Site-to-Site VPN을 복구(recovery) 용도로 사용

- **Replication**
  - RDS Replication (Cross Region), AWS Aurora + Global Databases
  - 온-프레미스에서 RDS로 Database replication
  - Storage Gateway

- **Automation**
  - CloudFormation / Elastic Beanstalk ~ 새로운 환경에서 새로 생성
  - CloudWatch로 alarm이 fail인 경우 EC2 인스턴스를 복구/재부팅
  - 커스터마이징된 자동화를 위해 AWS Lambda 함수를 사용

- **Chaos**
  - Netflix는 무작위로 EC2를 종료시켜버리는 [Chaos Monkey](https://github.com/netflix/chaosmonkey)를 보유하고 있음

## DMS - Database Migration Service

- 빠르고 안전하게 데이터베이스를 AWS로 마이그레이션
  - resilient
  - self healing
- source 데이터베이스는 마이그레이션 중에도 사용가능한 상태가 유지됨
- 지원:
  - Homogeneous migrations(동종 마이그레이션) ~ ex. Oracle to Oracle
  - Heterogeneous migrations(이기종 마이그레이션) ~ ex. Microsoft SQL Server to Aurora
- CDC를 통해 지속적인 Data Replication
- 반드시 replication 작업을 수행할 EC2 인스턴스를 생성해야 함

### DMS Sources and Targets

- **Sources**:
  - 온-프레미스 & EC2 인스턴스 DB
    - Oracle, MS SQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2
  - Azure: Azure SQL Database
  - Amazon RDS: Aurora 포함해서 전부
  - Amazon S3
  - DocumentDB
- **Targets**:
  - 온-프레미스 & EC2 인스턴스 DB
    - Oracle, MS SQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP
  - Amazon RDS
  - Redshift, DynamoDB, S3
  - OpenSearch Service
  - Kinesis Data Streams
  - Apache Kafka
  - DocumentDB & Amazon Neptune
  - Redis & Babelfish

## AWS Schema Conversion Tool (SCT)

- 한 엔진에서 다른 종류의 엔진으로 데이터베이스 스키마를 변환
  - ex. OLTP: (SQL Server or Oracle) to MySQL, PostgreSQL, Aurora
  - ex. OLAP: (Teradata or Orcale) to Amazon Redshift
- **동일한 DB 엔진으로 마이그레이션을 해야하는 상황이라면 SCT를 쓸 이유가 없음**
  - ex. 온-프레미스 PostgreSQL => RDS PostgreSQL
  - RDS는 플랫폼일 뿐, DB 엔진은 여전히 PostgreSQL

### DMS - Continuous Replication

![DMS Continous replication](https://velog.velcdn.com/images/chan9708/post/eef25414-eb76-48e2-95e7-46e6772ced69/image.png)

## RDS & Aurora MySQL Migrations

- RDS MySQL to Aurora MySQL
  - 옵션 1: RDS MySQL로부터 DB 스냅샷을 만들고 MySQL Aurora DB에서 복구
  - 옵션 2: RDS MySQL로부터 Aurora Read Replica를 만들고, replication lag이 0이 되었을 때, 이를 자체 DB 클러스터로 승격시킴 (시간, 비용 추가)
- 외부의 MySQL to Aurora MySQL
  - 옵션 1:
    - Percona XtraBackup으로 S3에서 파일 백업을 생성
    - S3로부터 Aurora MySQL DB를 생성
  - 옵션 2:
    - Aurora MySQL DB를 생성
    - mysqldump 유틸을 사용하여 MySQL을 Aurora로 마이그레이션 (S3를 통한 방법보다 느림)
- 만약 양측의 DB가 모두 실행 및 구동 중이라면 DMS를 사용

## RDS & Aurora postgreSQL Migrations

- RDS PostgreSQL to Aurora PostgreSQL
  - 옵션 1: RDS PostgreSQL로부터 DB 스냅샷을 만들고 PostgreSQL Aurora DB에서 복구
  - 옵션 2: RDS PostgreSQL로부터 Aurora Read Replica를 만들고, replication lag이 0이 되었을 때, 이를 자체 DB 클러스터로 승격시킴 (시간, 비용 추가)
- 외부의 PostgreSQL to Aurora PostgreSQL
  - 백업을 만들고 S3에 넣음
  - aws_s3 Aurora 익스텐션으로 이를 가져옴
- 만약 양측의 DB가 모두 실행 및 구동 중이라면 DMS를 사용

## On-Premise Strategy with AWS

- Amazon Linux 2 AMI를 VM으로 다운로드 할 수 있음 (.iso 포맷)
  - VMWare, KVM, VirtualBox (Oracle VM), Microsoft Hyper-V
- VM Import / Export
  - 기존 애플리케이션을 EC2로 마이그레이션
  - 온-프레미스 VM에 사용하기 위해 DR(Disaster Recovery) repo를 생성하는 전략
  - EC2에서 온-프레미스로 VM을 export할 수도 있음
- AWS Application Discovery Service
  - 마이그레이션 계획을 위해 온-프레미스 서버에 대한 정보를 수집
  - 서버 사용률 및 종속성 매핑
  - AWS Migration Hub로 트래킹
- AWS Database Migration Service (DMS)
  - 온-프레미스 => AWS, AWS => AWS, AWS => 온-프레미스 복제
  - 다양한 DB 엔진과 호환 (Oracle, MySQL, DynamoDB, etc...)
- AWS Server Migration Service (SMS)
  - 온-프레미스의 라이브 서버에서 AWS로의 증분 복제 (incremental replication)

## AWS Backup

- 완전 관리형 서비스
- AWS 서비스들의 백업을 중앙 관리 및 자동화
- 커스텀 스크립트를 생성하거나 수동으로 처리해야할 필요가 없음
- 지원 서비스:
  - EC2 / EBS
  - S3
  - RDS (모든 엔진 지원) / Aurora / DynamoDB
  - DocumentDB / Neptune
  - EFS / FSx (Lustre & Windows File Server)
  - AWS Storage Gateway (Volume Gateway)
- cross-region 백업 지원
- cross-account 백업 지원
- 지원 서비스에 대한 PITR(Point-In-Time Recovery) 지원
- On-Demand & Scheduled backup
- Tag-based backup 정책
- **Backup Plan**이라는 이름의 백업 정책 생성
  - 백업 주기 (매 12시간마다 / 매일 / 매주 / 매월 / cron 표현식 사용)
  - 백업 윈도우
  - Cold Storage로 전환 (Never / Days / Weeks / Months / Years)
  - 보존 기간 (Always / Days / Weeks / Months / Years)

### AWS Backup Vault Lock

- AWS Backup Vault에 저장한 백업들에 대해 WORM (Write Once Read Many) 상태를 강제
- 백업한 내용을 다음으로부터 보호하기 위한 추가 레이어:
  - 의도치 않거나, 악의적인 삭제 작업
  - 보존 기간을 더 짧게 하거나 변경하는 업데이트
- 이것이 활성화된 경우, 심지어 루트 이용자도 백업을 삭제할 수 없게 됨

## AWS Application Discovery Service

- 온-프레미스 데이터 센터에 대한 정보를 수집함으로써 프로젝트 마이그레이션을 계획
- 서버 사용률(server utilization)과 종속성 매핑은 마이그레이션에 있어 중요함

- **Agentless Discovery (AWS Agentless Discovery Connector)**
  - VM 인벤토리, 설정, 성능 히스토리 (CPU / 메모리 / 디스크 사용량)
- **Agent-based Discovery (AWS Application Discovery Agent)**
  - 시스템 구성, 시스템 성능, 실행 프로세스, 시스템 간의 네트워크 연결 세부사항
- 결과 데이터는 AWS Migration Hub로 볼 수 있음

## AWS Application Migration Service (MGN)

- AWS SMS(Server Migration Service)를 대체하는 ClodEndure Migration의 AWS 버전
- 애플리케이션을 AWS로 마이그레이션하는 작업을 간소화하는 Lift-and-Shift(rehost) 솔루션 서비스
- 물리적/가상/클라우드 서버들을 AWS에서 네이티브하게 실행되도록 변환해줌
- 넓은 범위의 플랫폼, OS, 데이터베이스를 지원
- 최소한의 downtime, 낮은 비용

![AWS MGN](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2021/04/15/2021-aws-mgn-how-it-works.jpg)

## Transferring large amount of data into AWS

- **사례**:
  - 클라우드로 200TB의 데이터를 전송하려고 함
  - 현재 100Mbps의 인터넷 연결을 보유
- **Over the internet / Site-to-Site VPN**:
  - 즉시 설정 가능
  - 200(TB)\*1000(GB)\*1000(MB)\*8(Mb)/100Mbps = 16,000,000s = 185일
- **Over direct connect 1Gbps**:
  - 한번 설정에 많은 시간이 걸림 (1달 이상)
  - 200(TB)\*1000(GB)\*8(GB)/1Gbps = 1,600,000s = 18.5일
- **Over Snowball**:
  - 2 ~ 3개의 Snowball을 병렬적으로 받음
  - E2E 전송에 약 1주일 소요 (제일 빠름)
  - DMS와 함께 사용할 수 있음
- **지속적인(on-going) 복제 / 전송의 경우**: Site-to-Site VPN 또는 DX 또는 DMS 또는 DataSync

## VMware Cloud on AWS

- 일부 고객들은 VMware Cloud를 사용해서 본인들의 온-프레미스 데이터 센터를 관리하고 싶어함
  - 데이터 센터 용량을 AWS로 확장하려고 하지만, 여전히 VMware Cloud 소프트웨어의 사용은 유지하고 싶은 경우?
  - VMware Cloud on AWS를 사용하면 됨!
- 사례
  - VMware vSphere 기반의 워크로드를 AWS로 마이그레이션하고 싶은 경우
  - 프로덕션 워크로드를 VMware vSphere 기반의 private/public/hybrid 클라우드 환경으로 실행하고 싶은 경우
  - 재해 복구 전략을 갖고자 하는 경우

![VMware Cloud on AWS](https://d1.awsstatic.com/Digital%20Marketing/Webinar/VMware/Product-Page-Diagram_VMware-Cloud-on-AWS%20(1).7e9542eb2ffb98892a81838ac2622fe340650824.png)
