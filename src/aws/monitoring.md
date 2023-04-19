# Monitoring & Audit

## Amazon CloudWatch Metrics

- CloudWatch는 AWS의 모든 서비스에 대한 metric를 제공함
- **Metric**은 모니터링할 변수를 의미 (CPUUtilization, Networkln...)
- metric은 **namespace**에 속하게 됨
- **Dimension**은 metric의 attribute (instance id, envrionment, etc...)
- 각 metric 당 최대 30 dimension
- metric은 **timestamps**를 가짐
- metric의 CloudWatch 대시보드를 만들 수 있음
- **CloudWatch Custom Metrics**를 만들 수 있음 (ex. RAM)

### CloudWatch Metric Streams

- 낮은 레이턴시로 원하는 목적지(destination)로 CloudWatch metric을 지속적으로 스트리밍할 수 있음 (**거의 실시간 전송**, 낮은 레이턴시)
  - Amazon Kinesis Data Firehose (=> 이후 목적지로)
  - 써드파티 서비스 제공자: Datadog, Dynatrace, New Relic, Splunk, Sumo Logic...
- 옵션을 통해 metric의 하위 집합만 스트리밍하도록 **metric을 필터링**할 수 있음

## CloudWatch Logs

- **Log groups**: 임의의 이름, 주로 애플리케이션
- **Log stream**: 애플리케이션 / 로그 파일 / 컨테이너 내 인스턴스
- 로그 만료 정책 (만기 X, 30일 등등)
- **CloudWatch Log는 다음으로 로그를 전송할 수 있음**:
  - Amazon S3 (내보내기)
  - Kinesis Data Streams
  - Kinesis Data Firehose
  - AWS Lambda
  - OpenSearch

### CloudWatch Logs - Sources

- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
- Elastic Beanstalk: 애플리케이션으로부터의 log 컬렉션
- ECS: 컨테이너로부터의 컬렉션
- AWS Lambda: 함수 로그로부터의 컬렉션
- VP Flow Logs: VPC 별 로그
- API Gateway
- 필터 기반의 CloudTrail
- Route53: DNS 쿼리 로그

### CloudWatch Logs - Metric Filters & Insights

- CloudWatch Logs는 필터 표현식(filter expressions)을 사용할 수 있음
  - ex. 한 로그 내 특정 IP를 찾아내기
  - ex. 로그 내에서 "ERROR" 발생 카운팅하기
- CloudWatch alarm을 트리거할 수 있도록 metric filter 사용
- CloudWatch Logs Insights는 로그를 쿼리하거나, CloudWatch Dashboard에 쿼리를 추가하는 데 사용될 수 있음

### CloudWatch Logs - S3 Export

- 로그 데이터는 내보내기가 가능해지기까지 **최대 12시간**까지 걸릴 수 있음
- API 호출은 **CreateExportTask**
- 실시간과는 거리가 멀기 때문에, 실시간 처리가 필요하다면 대신 Logs Subscription을 쓸 것

### CloudWatch Logs - Subscriptions

![CloudWatch Logs Subscriptions Example](https://cloudwellserved.com/wp-content/uploads/2020/06/kinesis1.png)

### CloudWatch Logs - for EC2

- 기본적으로, 로그는 EC2 머신에서 CloudWatch로 이동하지 않음
- EC2가 로그 파일을 전송하길 원한다면 CloudWatch agent를 써야함
- IAM 권한이 적절한지 확인 필요함
- CloudWatch log agent는 온-프레미스에서도 설정 가능

## CloudWatch Logs Agent & Unified Agent

- 가상 서버를 위한 것 (EC2 인스턴스, 온-프레미스 서버)
- **CloudWatch Logs Agent**
  - 구버전 agent
  - 오직 CloudWatch Log에 전송 가능
- **CloudWatch Unified Agent**
  - RAM, 프로세스 등 추가적인 시스템 레벨 metric도 수집
  - 로그를 수집하여 CloudWatch Log에 전송
  - SSM Parameter Store를 통해 중앙화된 설정

### CloudWatch Unified Agent - Metrics

- 리눅스 서버 / EC2 인스턴스에서 직접 수집
  - **CPU** (active, ugest, idle, system, user, steal)
  - **Disk metrics** (free, used, total), Disk IO (writes, reads, bytes, iops)
  - **RAM** (free, inactive, used, total, cached)
  - **Netstat** (TCP/UDP 연결 수, net packets, bytes)
  - **Processes** (total, dead, bloqued, idle, running, sleep)
  - **Swap Space** (free, used, used %)
- 기억할 것: EC2에서의 기본 제공 metric - disk, CPU, network (high level)

## CloudWatch Alarms

- Alarm은 어떤 metric에 대한 알림을 트리거하기 위해 사용
- 다양한 옵션 (샘플링, %, max, min, etc...)
- Alarm States:
  - `OK`
  - `INSUFFICIENT_DATA`
  - `ALARM`
- Period:
  - metric을 평가하는데 걸리는 초단위 시간
  - 고해상도 커스텀 metric: 10초, 30초, 60초의 배수

### CloudWatch Alarm - Targets

- EC2 인스턴스를 중지 / 종료 / 재부팅 / 복구
- 오토 스케일링 동작을 트리거
- SNS에 알림 전송 (lambda를 통해 거의 뭐든 처리 가능)

### CloudWatch Alarm - Composition Alarm

- CloudWatch Alarm은 하나의 metric에서 동작
- **Composition Alarm은 여러 개의 alarm들에 기반한 상태를 모니터링**
- AND 또는 OR 조건
- 복잡한 composite alarm을 생성하여 alarm noise를 줄이는데 도움을 줌

### CloudWatch Alarm - EC2 Instance Recovery

- **Status Check**:
  - Instance status = EC2 VM 체크
  - System status = 기반 하드웨어 체크
- **Recovery**: 동일한 private/public/elastic IP, 동일한 메타데이터, 동일한 placement group

### CloudWatch Alarm: good to know

- Alarm은 CloudWatch Logs Metrics Filter에 기반하여 생성될 수 있음
- alarm과 notification을 테스트해보기 위해 CLI를 사용해서 alarm state를 변경할 수 있음

> `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purposes"`

## Amazon EventBridge (formerly CloudWatch Events)

- Schedule: 크론잡 (Cron Jobs - scheduled scripts)
- Event Pattern: 이벤트 규칙을 통해 서비스가 수행하는 작업에 반응
- Lambda 함수 트리거, SQS/SNS 메시지 전송

### Amazon EventBridge Rules

- Source에서 이벤트가 발생 (ex. EC2 인스턴스 시작, S3 오브젝트 업로드 등..)
- 위의 이벤트들이 필터링(선택 사항)되어 EventBridge로 전달
- 전달된 이벤트는 JSON 형태가 되어 destination으로 전달

```json
{
  "version": "0",
  "id": "6a7e8feb-b491",
  "detail-type": "EC2 Instance State-change Notification",
  // ...
}
```

### Amazon EventBridge - Event Bus

- 몇가지 종류의 Event Bus
  - Default Event Bus - AWS Services
  - Partner Event Bus - AWS Saas Partners (ex. Zendesk, Datadog)
  - Custom Event Bus - Custom Apps
- event bus들은 Resource-based Policy를 통해 다른 AWS 계정들로부터도 액세스 가능함
- event bus에 전달된 이벤트들(전체 또는 필터)은 **아카이빙**할 수 있음 (무기한 또는 기간 설정)
- **아카이빙된 이벤트를 재실행**할 수 있는 기능

### Amazon EventBridge - Schema Registry

- EventBridge는 event bus 내에서 이벤트를 분석하여 **스키마(schema)**를 유추할 수 있음
- **Schema Registry**는 애플리케이션을 위해, 이벤트 버스 내에서 데이터가 어떻게 구성될지 미리 알 수 있게 해주는 코드를 생성할 수 있도록 함
- Schema는 버저닝될 수 있음

### Amazon EventBridge - Resource-based Policy

- 특정 Event Bus 내 권한을 관리
- 예시: 다른 AWS 계정 또는 AWS 리전으로부터의 이벤트를 허용/거부
- 사례: 내 AWS Organization에서 일어나는 모든 이벤트들을 하나의 AWS 계정 또는 AWS 리전으로 통합

## CloudWatch Insights

### CloudWatch Insights - Container

- 컨테이너로부터 **메트릭과 로그**를 수집/병합/요약
- 다음 서비스에서의 컨테이너에 적용 가능
  - Amazon ECS
  - Amazon EKS
  - Kubernetes platforms on EC2
  - Fargate (both for ECS and EKS)
- Amazon EKS와 Kubernetes의 경우, CloudWatch Insights는 컨테이너 검색을 위해 CloudWatch Agent의 컨테이너화 버전을 사용함

### CloudWatch Insights - Lambda

- AWS Lambda에서 동작하는 서버리스 애플리케이션에 대한 모니터링 / 트러블슈팅 솔루션
- CPU 시간, 메모리, 디스크, 네트워크를 포함한 시스템 레벨 메트릭을 수집/병합/요약
- 콜드 스타트, lambda 워커 종료와 같은 진단 정보(diagnostic information)도 수집/병합/요약
- Lambda Insights가 Lambda Layer로 제공됨

### CloudWatch Insights - Contributor

- 로그 데이터를 분석하여 contributor 데이터를 표시하는 시계열 데이터를 생성
  - **top-N contributor에 대한 메트릭**을 볼 수 있음
  - unique contributor의 전체 수와 그들의 사용 내역
- top talker를 찾고, 누가/어떤 것이 시스템 성능에 영향을 미치는지 이해하는데에 도움
- 어떤 AWS 생성 로그와도 호환 (VPC, DNS, etc...)
- 사례:
  - bad host 찾기
  - **제일 네트워크 사용량이 많은 이용자 찾기**
  - 가장 많은 에러가 발생하는 URL 찾기
- 룰을 처음부터 구축할 수도 있고, 혹은 AWS에서 만들어진 샘플 룰을 사용할 수도 있음 - **내 CloudWatch Log를 활용**
- CloudWatch는 또한 다른 AWS 서비스로부터의 메트릭을 분석하는데 사용할 수 있는 빌트인 룰을 제공함

### CloudWatch Insights - Application

- **모니터되고 있는 애플리케이션에서 예측되는 잠재적인 문제를 보여주고, 진행중인 문제를 격리하기 위한 도움을 주는 자동화된 대시보드를 제공**
- 일부 기술만 적용된 EC2 인스턴스에서 애플리케이션을 실행할 수 있음 (Java, .NET, Microsoft IIS Web Server, databases...)
- 또, 다른 AWS를 사용할 수 있음 (ex. Amazon EBS, RDS, ELB, ASG, Lambda, SQS, DynamoDB, S3 bucket, ECS, EKS, SNS, API Gateway...)
- SageMaker를 내부적으로 사용
- 애플리케이션 health에 대한 가시성을 향상시켜 트러블슈팅 및 애플리케이션 오류 수정에 걸리는 시간을 줄여줌
- Amazon EventBridge와 SSM OpsCenter에 발견된 사항과 경고를 전달할 수 있음

### CloudWatch Insights and Operatonal Visibility

- CloudWatch Container Insights
  - ECS, EKS, Kubernetes on EC2, Fargate, 쿠버네티스의 경우 agent 필요
  - Metrics와 logs
- CloudWatch Lambda Insights
  - 서버리스 애플리케이션을 트러블슈팅하기 위한 구체적인 메트릭
- CloudWatch Contributors Insights
  - CloudWatch Logs를 통해 "Top-N" 기여자를 발견
- CloudWatch Application Insights
  - 애플리케이션과, 그와 관련된 AWS 서비스를 트러블슈팅하기 위한 자동화된 대시보드

## AWS CloudTrail

- **AWS 계정에 대한 governance, compliance, audit을 제공**
- CloudTrail은 기본적으로 활성화
- 아래로부터 이루어진 **내 AWS 계정으로 이루어진 이벤트 / API 호출 내역**을 가져올 수 있음
  - 콘솔
  - SDK
  - CLI
  - AWS Services
- CloudTrail로부터 CloudWatch Log 또는 S3로 로그를 전달할 수 있음
- **모든 리전 (기본) 또는 단일 리전에 적용 가능**
- AWS에서 리소스가 삭제된 경우, 먼저 CloudTrail을 살펴볼 것!

### AWS CloudTrail - Events

- **Management Events**:
  - AWS 계정 내 리소스에 이루어진 작업
  - 예시:
    - 보안 설정 (IAM `AttachRolePolicy`)
    - 라우팅 데이터에 대한 룰 설정 (Amazon EC2 `CreateSubnet`)
    - 로깅 설정 (AWS CloudTrail `CreateTrail`)
  - **기본적으로, CloudTrail은 Management Events들을 로그하도록 설정되어 있음**
  - **Write Events**(리소스에 대한 수정)과 **Read Events**(리소스를 수정하지 않음)를 구분할 수 있음
- **Data Events**:
  - **기본적으로, Data Events들은 로그되지 않음** (**대용량 작업이기 때문**)
  - Amazon S3의 오브젝트 레벨 활동 (ex. `GetObject`, `DeleteObject`, `PutObject`): Read Events와 Write Event 구분 가능
  - AWS Lambda 함수 실행 활동 (`Invoke` API)
- **CloudTrail Insights Events**: 아래에서 설명

### AWS CloudTrail - Insights

- **CloudTrail Insight의 활성화로 계정 내 일어난 일반적이지 않은 활동을 감지할 수 있음**
  - 부정확한 리소스 프로비저닝
  - 서비스 한도 초과
  - AWS IAM 액션의 폭발적인 증가
  - 정기적인 유지보수 활동의 부재
- CloudTrail은 기준점을 잡기 위해 일반적인 management event들을 분석
- 이후, **일반적이지 않은 패턴을 탐색하기 위해 지속적으로 write event를 분석**
  - 이상 징후는 CloudTrail 콘솔 내에 나타남
  - 이벤트는 S3에 전달됨
  - EventBridge 이벤트가 생성됨 (자동화가 필요한 경우)

### AWS CloudTrail - Retention

- Event들은 CloudTrail 내에 90일 동안 보관됨
- 이벤트를 그 이상 보관하려면, S3에 이를 로깅 (이후 Athena를 통해 분석 가능)

## AWS Config

- AWS 리소스의 **compliance(규정)**를 준수하고 기록하도록 도와줌
- 시간에 따른 설정과 변경을 기록하는데 도움
- AWS Config으로 해결할 수 있는 것들
  - 내 Security group에 제한이 없는 SSH 액세스가 존재하는가?
  - 내 버킷에 public access가 존재하는가?
  - 시간이 지나면서 내 ALB 설정이 바뀐 적이 있는가?
- 어떤 변경이든 경고(SNS notifications)를 받도록 할 수 있음
- AWS Config은 리전 별 서비스
- 리전과 계정 간에 통합할 수도 있음
- 설정 데이터를 S3에 보관할 수 있음 (Athena로 분석될 수 있음)

### AWS Config - Config Rules

- AWS에 의해 관리되는 config rule을 사용할 수 있음 (75개 이상)
- 커스텀 config rule을 사용할 수 있음 (반드시 AWS Lambda로 정의되어야 함)
  - ex. 모든 EBS 디스크가 gp2 타입이 맞는지 체크
  - ex. 모든 EC2 인스턴스가 t2.micro가 맞는지 체크
- Rule 자체도 평가 및 트리거될 수 있음
  - config이 변경될 때마다
  - And / Or: 일정 시간 간격으로
- **AWS Config Rules는 어떤 동작이 발생하는 것 자체를 막진 않음 (no deny)**
- **Pricing**: 프리티어 없음, 리전 별로 기록된 configuration item마다 $0.003, 리전 별 config rule evaluation마다 $0.001

### AWS Config - Resource

- 시간 경과에 따른 리소스의 규정 확인
- 시간 경과에 따른 리소스 설정 확인
- 시간 경과에 따른 CloudTrail API 호출 확인

### AWS Config - Remediations

- SSM Automation Documents을 통해 비규격(non-compliant) 리소스에 대한 수정을 자동화
- AWS-Managed Automation Documents를 사용하거나 커스텀 Automation Documents를 생성
  - 팁: Lambda 함수를 실행하는 커스텀 Automation Documents를 만들 수 있음
- 만약 리소스가 auto-remediation이 수행된 이후에도 여전히 규정을 준수하지 않는 상태라면, **Remediation Retries**를 설정할 수 있음

### AWS Config - Notifications

- AWS 리소스가 비규격 상태일 때, EventBridge를 사용하여 notification을 트리거
- 설정 변화와 규정 준수에 대한 상태 알림을 SNS로 전송할 수 있음
  - 모든 이벤트가 전달되며, SNS 필터링 또는 클라이언트 사이드 필터링을 적용할 수 있음

## CloudWatch vs. CloudTrail vs. Config

- CloudWatch
  - 퍼포먼스 모니터링 (메트릭, CPU, 네트워크 등) & 대시보드
  - Events & Alerting
  - 로그 병합 & 분석
- CloudTrail
  - 계정 내 일어난 모든 API 호출을 기록
  - 특정 리소스에 대한 trail을 정의
  - 글로벌 서비스
- Config
  - 설정 변경을 기록
  - 리소스들의 규정(compliance) 준수 여부를 판단
  - 변경과 규정 준수에 대한 타임라인 제공

### CloudWatch vs. CloudTrail vs. Config - For an Elastic Load Balancer

- CloudWatch:
  - 들어오는(incoming) 연결 metric을 모니터링
  - 시간 경과에 따른 에러 코드 비율을 %로 시각화
  - 로드 밸런서 성능에 대한 아이디어를 얻기 위해 대시보드를 구축
- Config:
  - 로드 밸런서의 Security Group 룰을 트래킹
  - 로드 밸런서의 설정 변경을 트래킹
  - SSL 인증서가 항상 로드 밸런서에 할당되도록 보장 (compliance)
- CloudTrail:
  - API 호출을 통해 누가 로드 밸런서를 변경했는지 트래킹
