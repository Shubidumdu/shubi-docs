# AWS RDS

- RDS : Relational Database Service
- SQL을 쿼리 언어로 사용하는 DB를 위한 관리형 DB 서비스
- AWS를 통해 관리되는 클라우드에 데이터베이스를 생성할 수 있도록 해줌
  - Postgres
  - MySql
  - MariaDB
  - Oracle
  - Microsoft SQL Server
  - Aurora (AWS Proprietary database)

## Advantage over using RDS versus deploying DB on EC2

- RDS는 관리형 서비스
  - 자동화된 프로비저닝(provisioning), OS 패칭(patching)
  - 지속적인 백업과 구체적인 타임스탬프로 복구 (Point in Time Restore)
  - 대쉬보드 모니터링
  - Read replica -> 읽기 성능 향상
  - Multi AZ setup for DR (Disaster Recovery)
  - Maintenance windows for upgrades
  - Scaling capability (vertical and horizontal)
  - 저장소를 EBS로 백업 (gp2 or io1)
- 인스턴스로 SSH 접속을 할 수는 없음

## RDS - Storage Auto Scaling

- RDS DB 인스턴스의 가용량(storage)를 동적으로 상승시킬 수 있도록 도와줌
- RDS가 남은 데이터베이스의 가용량이 고갈됨을 확인할 때, 자동으로 스케일을 확장
- DB 스토리지를 수동으로 확장하지 않아도 됨
- **Maximum Storage Threshold** (DB 스토리지의 최대 한계값)을 설정해야 함
- 다음의 경우들에 스토리지는 자동으로 수정됨
  - 할당된 스토리지의 10%보다 남은 가용량이 적은 경우
  - 낮은 가용량이 최소 5분 동안 지속되는 경우
  - 최근의 수정으로부터 6시간이 지난 이후
- **예측 불가능한 워크로드(unpredictable workloads)** 를 가진 애플리케이션에 유용함.
- 모든 RDS DB 엔진에 대해 지원 (MariaDB, MySQL, PostgreSQL, SQL Server, Oracle)

## RDS Read Replicas for read scalability

- 5개의 Read Replicas 까지 지원
- Within AZ, Cross AZ 또는 Cross Region (세가지 옵션)
- Replication은 **비동기적**(ASYNC), 따라서 읽기 작업이 결국 일관적(consistent)이다.
- Replica는 그들 자신의 DB로 승격(promote)될 수 있다.
- 애플리케이션들은 read replica들을 사용하고자 하는 경우 connection string을 업데이트해야만 한다.

## RDS Read Replicas - Use Cases

- 일반적인 load를 처리하는 프로덕션 DB를 보유한 경우
- 분석을 위해 리포팅을 하는 애플리케이션을 실행하고 싶다면
- 새로운 workload를 처리하는 Read Replica를 만듬
- 이 때, 프로덕션 애플리케이션에는 영향이 가지 않음
- Read Replica는 오직 SELECT(=read) 관련문만 처리할 수 있음 (not INSERT, UPDATE, DELETE)

## RDS Read Replicas - Network Cost

- AWS에서는 하나의 AZ에서 다른 곳으로 전달되는 데이터의 경우에는 network cost가 발생함
- **동일한 region 내에 있는 RDS Read Replica의 경우에는 비용이 부과되지 않음**

## RDS Multi AZ (Disaster Recovery)

- SYNC replication
- 하나의 DNS 네임 -> automatic app failover 대비
- **availability** 향상
- AZ의 손실, Network 손실, 인스턴스 또는 스토리지의 실패에 대한 장애조치
- 앱에 대한 어떤 수동적인 개입(intervention)을 하지 않음
- 확장을 위해 사용되지 않음
- **중요**: **Read Replica는 Disaster Recovery(DR)을 위해 Multi AZ로 셋업될 수 있다.**

### RDS - From Single-AZ to Multi-AZ

- Zero downtime operation (DB를 멈출 필요 없음)
- 단순히 DB에 대한 "수정(modify)" 버튼을 클릭하기만 하면 됨
- 그러면 아래와 같은 작업들이 내부적으로 일어남
  - 스냅샷이 찍힘
  - 새 DB가 새로운 AZ에서 앞서 찍은 스냅샷을 통해 복구됨.
  - 두 DB 사이에 동기화(synchronization) 작업이 이루어짐.

## RDS Custom

- **OS와 DB에 대한 커스터마이징이 가능한 관리형 Oracle 또는 Microsoft SQL Server 데이터베이스**
- RDS: AWS 내 DB의 설정, 작업, 스케일링의 자동화
- Custom: 그 아래 놓인 DB와 OS에 대한 접근으로, 다음과 같은 일들을 할 수 있음.
  - Configure settings
  - Install patches
  - Enable native features
  - **SSH 또는 SSM Session Manager**를 통해 RDS 아래의 EC2 인스턴스에 접근
- 커스터마이징을 하려면 **Automation Mode**를 비활성화 해야함 -> 그 전에 DB 스냅샷을 찍어두는 것을 권장
- RDS vs. RDS Custom
  - RDS: AWS로부터 관리되는 DB와 OS 전체
  - RDS Custom: 그 아래 놓인 OS와 DB에 대해 완전한 어드민 접근

## Amazon Aurora

- Aurora는 AWS가 소유한(proprietary) 자체적인 기술 (오픈 소스가 아님)
- Postgres와 MySQL 모두 Aurora DB와 호환됨 (= Aurora를 Postgres나 MySQL DB처럼 사용할 수 있음)
- Aurora는 AWS 클라우드에 최적화되어 있어, RDS의 MySQL보다 5배, RDS의 Postgres보다 3배 더 높은 성능 향상을 기대할 수 있음
- Aurora 스토리지는 자동으로 10GB까지 증가될 수 있으며, 최대 128TB까지.
- Aurora는 15개의 replica를 가질 수 있음. (MySQL은 5개까지) 또한 이러한 replication 과정이 빠름 (10ms 미만의 replica lag)
- Aurora의 장애조치(Failover)는 즉각적(instantaneous)이며, High-Availity(HA) native임.
- Aurora는 RDS보다 높은 가격(20% 이상)이지만, 더 효율적임.

### Aurora High Availability and Read Scaling

- 3개의 AZ에 걸쳐 6개의 데이터 사본이 생성됨
  - 쓰기 시 6개 중 4개 필요
  - 읽기 시 6개 중 3개 필요
  - peer-to-peer 복제를 통한 자가 복구
  - 스토리지가 100개의 볼륨에 걸쳐 스트라이프 처리
- 하나의 Aurora 인스턴스가 쓰기 작업을 처리 (master)
- 마스터에 대한 장애조치는 30초 이내로 처리됨
- master에 더해, 최대 15개의 Aurora Read Replicas가 읽기 작업을 처리
- Cross Region Replication을 지원함

### Aurora DB Cluster

![Aurora Cluster](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2017/11/01/AZ-Cluster-Endpoint.jpg)

### Features of Aurora

- 자동 장애조치
- 백업 및 복구
- 격리(isolation) 및 보안
- 산업 규정 준수 (industry compliance)
- Push-button scaling
- Zero downtime으로 자동화된 패칭(Patching)
- 고급 모니터링(Advanced Monitoring)
- Routine maintenace
- 백트래킹(Backtrack): 백업 없이 특정한 시점으로 데이터를 복구

### Aurora Replicas - Auto Scaling

![Aurora auto scaling](https://velog.velcdn.com/images%2Fcombi_jihoon%2Fpost%2F79a5fe80-4bb1-4883-a750-62b40d4d2c69%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-03-07%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.38.10.png)

### Aurora - Custom Endpoints

![Aurora custom endpoint](https://miro.medium.com/v2/resize:fit:1200/1*VC6IYPGQ_JR4ZDam1VktJw.png)

- Aurora Instance의 부분 집합을 커스텀 엔드포인트로 정의
- Ex. 특정 replica(사본)들에서만 분석 쿼리를 실행
- Custom Endpoint의 정의 이후에 일반적으로 Reader Endpoint는 사용되지 않음.

### Aurora Serverless

- 실제 사용에 기반한 자동화된 데이터베이스 인스턴스화(Instantiation)와 오토 스케일링
- 비주기적(infrequent)이고, 간헐적(intermittent)이며, 예측 불가능한(unpredictable) 워크로드의 경우에 유용함
- capacity planning(특정 capacity를 선택해야 할 필요)가없음
- 매초마다 비용을 지불하며, 더 비용 효율적임(cost-effective).
  
### Aurora Multi-Master

- write node 에 대한 즉각적인 failover를 원할 때 (high availability)
- 모든 노드가 R/W를 수행 - vs. 하나의 Read replica를 새로운 master로 승격

### Global Aurora

- Aurora Cross Region Read Replicas:
  - disaster recovery(장애 조치)에 유용
  - 설치가 간편함
- Aurora Global Database (recommended):
  - 하나의 주요 리전 (Read / Write)
  - 5개 까지의 보조 리전(Read 전용), 복제에 걸리는 지연 시간이 1초 미만임
  - 각 보조 리전 당 16개까지의 Read Replica
  - latency 감소에 도움
  - 또다른 리전 승격(장애 조치 목적)은 RTO(= Recovery Time Objective)가 1분 미만
  - **일반적인 cross-region 복제는 1초 미만이 소요됨**

### Aurora Machine Learning

![Aurora Machin Learning](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2019/11/23/aurora-ml-integrations.png)

- SQL을 통해 애플리케이션에 대한 ML 기반 예측 활성화
- 간단하고, 최적화되어 있으며, 안전하게 통합된 Aurora와 AWS ML 서비스
- 지원되는 서비스
  - Amazon SageMaker (어떤 ML 모델도 사용 가능)
  - Amazon Comprehend (감성 분석 용도)
- 굳이 ML 경험을 필요로 하지 않음
- 이용 사례: 사기 감지(fraud detection), 광고 타겟팅(ads targeting), 감성 분석(sentiment analysis), 제품 추천(product recommendation)
