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
