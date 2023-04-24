# More Solution Architectures

## Event Processing in AWS

### Lambda, SNS & SQS

- SQS + Lambda
  - SQS에서 lambda로 poll을 시도하다가, 지속적으로 실패할 경우 무한 루프에 빠질 수 있음
  - 이에 따라 DLQ(Dead Letter Queue)를 설정할 수 있음 (ex. 5번 retry 후 DLQ로 이동)
- SQS FIFO + Lambda
  - SQS FIFO의 경우, 순서를 보장하기 때문에, 한번 실패할 경우, 그 다음의 메시지로 넘어가지 않고 블록된 상태가 되버림
  - 이때도 마찬가지로 DLQ를 설정하여 다음 메시지로 넘어갈 수 있도록 대처할 수 있음
- SNS + Lambda
  - SNS에서 Lambda로 비동기적으로 메시지를 전송하였으나 Lambda 함수에서의 처리가 이루어지지 않을 때
  - 마찬가지로 몇번의 retry를 시도한 다음, 버리거나(discard), DLQ를 설정하여 SQS로 전달할 수 있음 (이 때는, Lambda 서비스 레벨에서 DLQ를 설정했다는 점에서 앞의 두개와는 차이가 있음)

### Fan Out Pattern: deliver to multiple SQS

- 여러 개의 SQS를 둔 경우, 각각의 SQS에다 직접 Message를 PUT할 수도 있으나, 이 경우 아키덱처를 신뢰하기 어려워짐
  - 왜? -> x번째 메시지를 처리하던 중 애플리케이션에 문제가 발생한다면, 그 이후의 메시지는 처리되지 않아 끝내 못받게되는 경우가 발생할 수 있음
  - 이 때는 **Fan Out** 패턴을 쓰자!
  - 하나의 SNS를 두고, 이를 여러 SQS가 구독하는 형태

### S3 Event Notifications

- `S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication`...
- SQS, SNS, Lambda와 연동하여 사용
- 오브젝트 명칭 필터링이 가능 (ex. `*.jpg`)
- 사례: S3에 업로드된 이미지의 썸네일 생성
- **원하는 만큼 많이 "S3 이벤트"를 만들 수 있음**
- S3 Event Notifications는 일반적으로 몇 초 안에 전달되지만, 경우에 따라서는 몇 분 이상 소요될 경우도 있음

### S3 Event Notifications with Amazon EventBridge

- JSON Rule을 통한 **고급 필터링** 옵션 (메타데이터, 오브젝트 사이즈, 이름...)
- **Multiple Destinations** - ex. Step Functions, Kinesis Streams / Firehose...
- **EventBridge Capabilities** - Archive, Replay Events, Reliable delivery

![S3 Event Notifications with Amazon EventBridge](https://da-public-assets.s3.amazonaws.com/thumbnails/patterns/s3-eventbridge-sns-terraform.png)

### Amazon EventBridge - Intercept API Calls

- DynamoDB -> CloudTrail -> EventBridge -> SNS

### API Gateway - AWS Service Integration Kinesis Data Streams Example

- Client -> API Gateway -> Kinesis Data Streams -> Kinesis Data Firehose -> Amazon S3

## Caching Strategies

![AWS Caching Strategies](https://blog.cloudcraft.co/wp-content/uploads/2020/09/Caching-101-1.png)

## Blocking an IP address

- 가장 쉬운 방법은 NACL(Network Access Control List)을 통해 VPC 레벨에서 원하는 IP 주소에 대해 Deny Rule을 설정 하는 것
  - 여기에 EC2 내에 선택적으로 Firewall 소프트웨어를 사용할 수도 있음
- ALB의 경우:
  - NACL + ALB 측에서 Security Group을 설정 -> Connection Termination
- NLB의 경우:
  - Connection Termination이 없음 -> 곧바로 EC2 인스턴스로 넘어감
- ALB + WAF:
  - WAF에서, 좀 더 비싸지만 복잡한 필터링을 구축할 수 있음
  - WAF는 ALB에다 구축하는 것
- ALB + CloudFront WAF:
  - CloudFront 쪽에서 WAF를 이용해 IP 주소를 먼저 필터링
  - 이후, ALB로 넘기는데, 이때는 CloudFront의 Public IP를 통해 접속하는 것이라, 클라이언트의 IP를 알 수가 없어 NACL이 도움이 되지 않음

## High Performance Computing (HPC)

- 클라우드는 HPC를 수행하기에 완벽한 곳임
- 언제든지 엄청 많은 수의 리소스들을 생성할 수 있음
- 더 많은 리소스들을 추가함으로써 결과물의 속도를 높일 수 있음
- 사용한 시스템 만큼만 값을 지불
- Perform genomics, computational chemistry, financial risk modeling, weather prediction, machine learning, deep learning, autonomous driving
- HPC를 수행하려면 어떤 서비스를 사용해야 할까?

### Data Management & Transfer

- AWS Direct Connect
  - private 보안 네트워크를 통해 클라우드에 GB/s 수준의 데이터를 이동
- Snowball & Snowmobile
  - 클라우드에 PB 단위의 데이터 이동
- AWS DataSync
  - 온-프레미스와 S3, EFS, FSx for Windows 간에 많은 양의 데이터를 이동

### Compute and Networking

- EC2 Instances:
  - CPU optimized, GPU optimized
  - Spot Instances / Spot Fleets for cost savings + Auto Scaling
- EC2 Placement Groups: 더 좋은 네트워크 성능을 위한 **클러스터**
- EC2 Enhanced Networking (SR-IOV)
  - 높은 대역폭, 높은 PPS (Packer Per Second), 낮은 레이턴시
  - 옵션 1: **Elastic Network Adapter (ENA)** ~ 최대 100Gbps
  - 옵션 2: Intel 82599 VF ~ 최대 10Gbps (레거시)
- **Elastic Fabric Adapter (EFA)**
  - HPA 목적의 향상된 ENA, **Linux**에서만 동작
  - 노드 간의 커뮤니케이션, **강하게 커플링된 워크로드**에 유용함
  - MPI(Message Passing Interface) 표준 활용
  - 기본 Linux OS를 우회하여, 낮은 레이턴시와 안정적인 전송을 제공

### Storage

- Instance-attached storage:
  - **EBS**: io2 Block Express 사용 시 최대 256,000 IOPS까지 스케일 업
  - **Instance Store**: million(백만) 단위 IOPS까지 스케일링, EC2 인스턴스에 연결, 낮은 레이턴시
- Network Storage:
  - **Amazon S3**: large blob, 파일 시스템이 아님
  - **Amazon EFS**: 전체 사이즈에 기반하여 IOPS가 스케일링, 또는 IOPS 프로비저닝
  - **Amazon FSx for Lustre**:
    - HPC 최적화된 분산형 파일 시스템, millions of IOPS
    - Backed by S3

### Automation and Orchestration

- AWS Batch
  - **AWS Batch**는 멀티 노트 병렬 작업(여러 **EC2** 인스턴스에 걸쳐 단일 작업을 처리)을 지원
  - 작업을 스케줄링하고, 이에 따른 EC2 인스턴스 실행을 쉽게 할 수 있음
- AWS ParallelCluster
  - AWS에서 HPC를 배포하기 위한 오픈소스 클러스터 관리 툴
  - 텍스트 파일로 설정
  - VPC, 서브넷, 클러스터 타입 및 인스턴스 타입 생성을 자동화
  - **클러스터에서 EFA를 활성화하는 기능** (**네트워크 성능 향상**)

## High Availability

### Creating a highly Available EC2 Instance

- Highly Available한 인스턴스를 직접 구성하려면
  - Elastic IP
  - 두 개의 EC2 인스턴스
    - Public EC2 (기본)
    - Standby EC2 (대기 용도)
  - CloudWatch Event (인스턴스 상태 모니터링 ~ metric에 따라 알람 전달)
  - Lambda (알람이 일어나면, 대기하던 인스턴스를 실행하고, Elastic IP를 할당)

### Creating a highly available EC2 instance With an Auto Scaling Group

- **ASG에서 스케일링 설정**
  - ex. 1min, 1max, 1desired, >= 2AZ
- Elastic IP를 (Tag에 기반하여) 새 인스턴스에 할당하는 EC2 user data를 세팅
  - EC2 인스턴스 Role이 Elastic IP를 할당하는 API 호출을 허용하고 있어야 함
- 기존 인스턴스는 제거

### Creating a highly available EC2 instance with ASG + EBS

- ASG에서 Terminate lifecycle hook으로 EC2 인스턴스가 종료되면서 EBS를 스냅샷 (+ 태깅)
- 이후 새 인스턴스를 실행하면서 (Tag에 기반한) EC2 user data로 EBS 볼륨을 생성하고 ASG Launch lifecycle hook에서 생성한 EBS 볼륨을 인스턴스에 연결
