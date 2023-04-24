# Other Services

## CloudFormation

- 어떤 리소스들(거의 대부분 지원)에 대해 AWS 인프라를 개괄적으로(declarative) 설명하는 선언적인 방법
- 예를 들어, CloudFormation 템플릿에서는 다음과 같은 것을 정의할 수 있음
  - Security group
  - Two EC2 instances using this security group
  - S3 Bucket
  - Load Balancer in front of these machines
- 그러면 CloudFormation에서는 이들을 **올바른 순서대로**, 내가 **정의한 설정과 똑같이** 리소스들을 생성해줌

### Benefits of AWS CloudFormation

- **코드로 인프라를 작성할 수 있음**
  - 어떤 리소스도 수동으로 생성할 필요 없음 -> 관리하기 편함
  - 인프라에 대한 변경사항이 코드를 통해 리뷰될 수 있음
- 비용
  - 스택에 포함된 각각의 리소스들은  별도의 식별자로 태그되며, 각 스택의 비용이 어떻게 되는지 쉽게 파악할 수 있음
  - CloudFormation 템플릿으로 리소스들의 비용을 추정할 수 있음
  - Savings Strategy:
    - ex. Dev 환경에서, 5PM에 템플릿을 자동 삭제 / 8AM에 재생성
- 생산성
  - 클라우드에서 즉시 사용 가능한 인프라를 생성 / 삭제 가능
  - 템플릿에 대한 다이어그램 자동 생성
  - 선언적 프로그래밍 (순서 / 오케스트레이션을 파악할 필요 없음)
- Don't re-invent the wheel
  - 웹에 존재하는 템플릿을 활용
  - document를 활용
- **(거의) 모든 AWS 리소스들을 지원**:
  - (적어도 여기에 정리했던) 모든 리소스들을 지원함
  - 지원되지 않는 리소스들을 위해서 "커스텀 리소스"를 사용할 수 있음

### CloudFormation Stack Designer

- 예시: WordPress CloudFormation Stack
- 모든 **리소스**들을 볼 수 있음
- 각 컴포넌트 간 **관계**를 파악할 수 있음

## Amazon SES (Simple Email Service)

- **규모에 따라 이메일을 안전하게, 글로벌로 보낼 수 있는 완전 관리형 서비스**
- 인바운드/아웃바운드 이메일 허용
- 평판 대시보드, 성능 인사이트, 안티-스팸 피드백
- 이메일 전송, 이메일 바운스, 피드백 루프 결과, 이메일 확인 여부 등의 통계 제공
- DKIM(DomainKeys Identified Mail)과 SPF(Sender Policy Framework) 지원
- Flexible IP 배포: shared / dedicated / customer-owned IP
- AWS 콘솔/API 또는 SMTP를 이용해 애플리케이션에서 메일을 전송할 수 있음
- 사례: 트랜잭션, 마케팅 및 대량 이메일 커뮤니케이션

## Amazon Pinpoint

- 확장 가능한(scalable) **2-way (아웃바운드/인바운드)** 마케팅 커뮤니케이션 서비스
- 이메일, SMS, 푸시, 보이스, 인앱 마케팅 지원
- 고객들에게 올바른 컨텐츠를 제공하기 위해 메시지를 세분화 및 개인화할 수 있음
- 응답 받기 가능
- 매일 최대 10억(billion)개의 메시지까지 스케일링 가능
- 사례: 마케팅/대량/거래 SMS 메시지 전송을 통한 캠페인 진행
- **Amazon SNS나 Amazon SES와의 차이점**
  - SNS & SES에서는 각 메시지의 수신자, 컨텐츠, 전달 스케줄을 관리
  - Amazon Pinpoint에서는 메시지 템플릿, 전달 스케줄, 고도로 타겟팅된(highly-targeted) 세그먼트 및 전체 캠페인을 생성할 수 있음

## System Manager - SSM Session Manager

- 내 EC2와 온-프레미스 서버에 대한 Secure Shell 연결을 할 수 있도록 해줌

- **SSH 액세스나 Bastion Host, SSH Key 필요 없음**
- **22번 포트 필요 없음 (높은 보안)**
- Linux, macOS, Windows 지원
- S3나 CloudWatch Logs에 세션 로그 데이터를 전송할 수 있음

### System Manager - Run Command

- document(= script)를 실행하거나 그냥 커맨드를 실행할 수 있음
- 여러 인스턴스 간에 커맨드를 실행 (리소스 그룹 사용 시)
- SSH 필요 없음
- AWS 콘솔을 통해 커맨드 결과를 볼 수 있고, S3 버킷 또는 CloudWatch Log로 전송
- 커맨드의 상태(실행 중, 완료, 실패)에 대해 SNS에 알림으로 보낼 수 있음
- IAM & CloudTrail과 호환됨
- EventBridge를 통해서도 실행될 수 있음

### System Manager - Patch Manager

- 관리 중인 인스턴스에 대한 패치(patch) 과정을 자동화
  - OS 업데이트, 애플리케이션 업데이트, 보안 업데이트 등
- EC2 인스턴스 및 온-프레미스 서버 지원
- Linux, macOS, Windows 지원
- 온-디맨드로 패치하거나, **Maintenance Windows**를 통해 패치를 스케줄링
- 인스턴스 스캔하여 패치 compliance 리포트를 생성 (missing patches)

### System Manager - Maintenance Windows

- 내 인스턴스에 액션을 처리할 시점에 대한 스케줄을 정의
  - ex. OS 패칭, 드라이버 업데이트, 소프트웨어 설치 등..
- Maintenance Window는 다음을 포함
  - Schedule
  - Duration
  - Set of registered instances
  - Set of registered tasks

### System Manager - Automation

- EC2 인스턴스와 다른 AWS 리소스들의 일반적인 유지보수와 배포 작업을 단순화시켜줌
  - ex. 인스턴스 재시작, AMI 생성, EBS 스냅샷
- **Automation Runbook** - 내 EC2 인스턴스나 AWS 리소스에 수행될 액션을 정의하는 SSM Document (pre-defined 또는 custom)
- 다음과 같은 것들을 통해 트리거될 수 있음
  - AWS 콘솔/CLI/SDK로 수동 트리거
  - Amazon EventBridge
  - Maintenance Window로 스케줄링
  - AWS Config (Rule을 수정하고자 할 때)

## Cost Explorer

- AWS 비용과 사용 시간에 대한 시각화 / 이해 / 관리
- 비용 및 사용 데이터에 대해 분석하는 커스텀 리포트를 생성할 수 있음
- 고수준(high-level)에서 데이터를 분석: 전체 계정에서의 전체 비용 및 사용
- 월/시간/리소스 단위로 세분화
- 최적의 **Savings Plan** 선택 (요금 청구 가격을 더 낮추고자 할 때)
- **이전 사용량을 기준으로 최대 12개월까지의 사용량 예측**

## Amazon Elastic Transcoder

- Elastic Transcoder는 **S3에 저장된 미디어 파일들을 소비자들의 재생 디바이스(ex. 휴대폰)에서 요구하는 포맷으로 변환해줌**
- 이점:
  - 사용하기 쉬움
  - Highly scalable - 많은 양의 미디어 파일 & 큰 사이즈의 파일을 다룰 수 있음
  - 비용 효율적 - duration 기반의 비용 모델
  - 완전 관리형 & 안전함, 쓴만큼 비용 지불 (온-디맨드)

## AWS Batch

- **규모에 관계 없는** **완전 관리형** 배치 프로세싱
- AWS에서 100,000개까지의 batch job들을 효율적으로 실행
- batch job이란?
  - 시작과 끝이 있는 작업 (continous와 반대됨)
- Batch는 **EC2 인스턴스** 또는 **Spot Instances**를 동적으로 실행함
- AWS Batch는 적절한 양의 컴퓨팅과 메모리를 프로비저닝함
- batch job을 제출 또는 스케줄링하기만 하면, 나머지는 AWS Batch가 알아서 처리함
- Batch job은 **Docker images**로 정의되고, **ECS에서 실행**
- 비용 최적화 & 인프라에 신경을 덜 써도 되도록 도와줌

### Batch vs Lambda

- Lambda:
  - 시간 제한 있음
  - 제한된 런타임
  - 제한된 임시 디스크 공간
  - 서버리스
- Batch:
  - 시간 제한 없음
  - Docker image로 패키징되기만 하면 어떤 런타임이든 가능
  - 디스크 공간의 경우 EBS / instance store에 의존
  - EC2에 의존 (AWS에 의해 관리될 수 있음)

## Amazon AppFlow

- **Software-as-a-Service (SaaS) 애플리케이션과 AWS 사이**의 안전한 데이터 전송을 할 수 있게 해주는 완전 관리형 통합 서비스
- **Sources**: **Salesforce**, SAP, Zendesk, Slack, ServiceNow
- **Destinations**: **Amazon S3, Amazon Redshift**와 같은 AWS 서비스 또는 SnowFlak, Salesforce 같은 non-AWS 서비스
- **Frequency**:
  - 스케줄을 만들거나
  - 이벤트에 대한 응답으로 처리되거나
  - 온-디맨드로 직접 수행
- **Data transformation**: 필터링 또는 validation과 같은 기능
- public 인터넷 상으로 암호화 또는 AWS PrivateLink를 통해 private하게 암호화
- integration 작성에 시간을 소요하지 않고, 바로 API를 활용
