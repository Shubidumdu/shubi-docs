# Decoupling Applications

- 여러 개의 애플리케이션을 배포하기 시작할 때, 필연적으로 애플리케이션들이 서로 소통해야할 필요가 생김
- 이 때, 애플리케이션 간의 커뮤니케이션을 구현하는데 있어 두 가지 패턴이 있음
  1. 동기 커뮤니케이션 (Synchronous communications ~ application to application)
  2. 비동기 / 이벤트 기반 커뮤니케이션 (Asynchronous / Event based ~ application to queue to application)
- 애플리케이션 간의 동기적인 커뮤니케이션은 갑작스러운 트래픽 급증이 있는 상황에서 문제가 될 수 있음
  - 만약, 일반적으로 10개 정도로 비디오 인코딩 작업을 처리하던 애플리케이션에서 갑자기 1000개의 요청을 처리하게 되면?
- 이러한 경우에 대비하기 위해, 애플리케이션은 **디커플링(decouple)**을 하는 것이 좋음
  - SQS: queue 모델
  - SNS: pub/sub 모델
  - Kinesis: 실시간 스트리밍 모델
- 위의 서비스들은 애플리케이션과는 독립적으로 스케일링을 수행할 수 있음

## SQS

### What's a queue?

![SQS queue](https://hands-on.cloud/wp-content/uploads/2023/01/A-Quick-Intro-To-Amazon-Simple-Queue-Service-SQS-Processing-messages-1024x451.png?ezimgfmt=rs:372x164/rscb1/ngcb1/notWebP)

### SQS - Standard Queue

- 오래된 서비스 (거의 10년 가까이)
- 주로 **애플리케이션 디커플링**에 사용되는 완전 관리형 서비스
- 속성:
  - 무제한 처리량(throughput), 큐 내 메시지 갯수에 제한이 없음
  - 메시지의 기본 보관 기관: 4일, 최대 14일
  - 낮은 레이턴시 (publish / receive에 10ms 미만)
  - 각 메시지 전송에 256kb 제한
- 메시지 중복이 발생할 수 있음 (At-least-once delivery에 의해, 가끔 발생할 수 있음)
- 메시지 순서가 올바르지 않을 수 있음 (Best-effort message ~ 메시지 순서를 최대한 지키려 노력하긴 하지만, 보장하진 않음)

### SQS - Producing Messages

- SDK를 사용하여 SQS로 메시지 생산 (SendMessage API)
- 해당 메시지는 consumer가 삭제하기 전까지 SQS 내에 지속됨
- 메시지 보관 기간: 기본 4일, 최대 14일
- 예시: 처리해야할 주문을 보내는 경우
  - Order id
  - Customer id
  - Any attributes
- SQS standard: 무제한 처리량(throughput)

### SQS - Consuming Messages

- Consumers (EC2 인스턴스, 서버 또는 AWS Lambda)
- 메시지를 받기 위한 SQS 폴링(Poll) - 한번에 최대 10개의 메시지 수신
- 메시지 처리 (ex. 받은 메시지를 RDS DB로 insert)
- DeleteMessage API를 사용하여 메시지 삭제

### SQS - Multiple EC2 Instances Consumers

- Consumer들은 메시지 수신/처리를 병렬적으로 수행
- At-least-once delivery
- Best-effort message ordering
- Consumer들은 메시지를 처리한 이후에는 삭제함
- 메시지 처리량(throughput)을 향상시키기 위해 Consumer를 수평적으로 스케일링할 수 있음

### SQS - with Auto Scaling Group (ASG)

![SQS with ASG](https://velog.velcdn.com/images/combi_jihoon/post/9f35b7bb-15a1-47e5-8739-2e117a5d7cd5/image.png)

- SQS 큐에 있는 메시지 갯수에 따라 CloudWatch 알람을 통해 ASG로 적절히 스케일링 할 수 있음

![Transaction Can be lost](https://miro.medium.com/v2/resize:fit:1400/1*APII-Xi0GFGm1VAzmy1X-A.png)

- 애플리케이션에 과부하가 온다면 일부 트랜잭션이 손실될 수 있음

![SQS as a buffer to database writes](https://miro.medium.com/v2/resize:fit:1400/1*doE7co618SwwqE3ANr9HFQ.png)

- SQS 큐를 버퍼로 둘 때, 이는 무한하게 확장 가능하기 때문에, 어떻게든 트랜잭션이 결국 처리된다는 것을 보장할 수 있음
- 다만, 클라이언트에게 트랜잭션 결과를 바로 보여주어야 하는 경우엔 부적합할 수 있음

### SQS - **decouple** between application tiers

![Decouple between application tiers](https://miro.medium.com/v2/resize:fit:1400/1*5JW1sSEyd0eotW-XBY_B7Q.png)

![Decouple messaging pattern](https://docs.aws.amazon.com/images/prescriptive-guidance/latest/modernization-integrating-microservices/images/integrating-diagram3.png)

### SQS - Security

- **Encryption**:
  - HTTPS API를 이용한 in-flight 암호화
  - KMS 키를 이용한 At-rest 암호화
  - 클라이언트가 자체적인 암호화/복호화를 수행하고자 원한다면 클라이언트 측 암호화도 가능
- **Access Controls**: SQS API에 대한 액세스를 조정하는 IAM 정책
- **SQS Access Policies** (S3 버킷 정책과 유사)
  - 서로 다른 계정 간의 SQS 큐에 액세스해야 하는 경우 유용함
  - 다른 서비스 (SNS, S3...)가 SQS 큐에 작성해야 하는 경우 유용함

### SQS - Message Visibility Timeout

![Message visibility timeout](https://docs.aws.amazon.com/images/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-visibility-timeout-diagram.png)

- 메시지가 consumer에 의해 폴링된 이후에는 다른 consumer들에게 **보이지 않는** 상태가 됨
- 기본적으로, "message visibility timeout"은 **30초**
  - 이는 즉, 메시지 처리를 30초 내에 해야한다는 의미
- message visibility timeout이 지난 이후에야, 메시지는 SQS에서 **보이는** 상태가 됨
- 만약, visibility timeout 내에 메시지가 처리되지 않는다면, 위의 사진 예시의 경우 해당 메시지는 **두번**씩 처리가 됨
- consumer는 이에 따라, 만약 visibility timeout보다 메시지 처리 시간이 더 걸린다면, **ChangeMessageVisibility API**를 호출하여 더 시간을 가질 수 있음
- visibility timeout이 너무 크다면 (hours), consumer에 문제가 발생했을 때, 재처리(re-processing)에 많은 시간이 걸림
- visibility timeout이 너무 작다면 (seconds), 중복 처리가 발생할 수 있음

### SQS - Long Polling

- consumer가 큐(queue ~ 대기열)로부터 메시지 요청을 보낼 때, 아직 큐에 있지 않은 메시지가 도착할 때까지 선택적으로 **기다릴** 수 있음
- 이를 Long Polling이라고 함
- **Long Polling은 SQS에 이루어지는 API 호출 횟수를 감소시켜, 애플리케이션의 효율성과 레이턴시를 개선함**
- wait time은 1초에서 20초 사이로 설정 가능 (20초 권장)
- Long Polling은 Short Polling보다 선호됨
- Long Polling은 큐 레벨에서 활성화되거나, **WaitTimeSeconds**를 통해 API 레벨에서 활성화될 수 있음

### SQS - FIFO Queue

![SQS FIFO](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/05/01/sqs_fifo_blog_img5.png)

- FIFO = First In First Out (큐 내 메시지의 순서)
- 제한적인 처리량(throughput): 배칭(batching)없이 300msg/s, 배칭있는 경우 3000msg/s
- Exactly-once send capability (중복 제거)
- Consumer의 처리 순서를 보장할 수 있음

## SNS

- 만약 하나의 메시지를 많은 receiver들에게 전달하고자 한다면 어떻게 해야 할까?
- 하나의 애플리케이션이 각각의 receiver들에게 직접 메시지를 전달하는 방법도 있겠지만, 이 경우, receiver가 추가될 때마다 일일이 애플리케이션 코드를 수정해야 하는 문제가 발생
- 이를 해결하기 위한 방법이 Pub/Sub(게시/구독) 패턴

![Pub/Sub Pattern with SNS](https://d1.awsstatic.com/Product-Page-Diagram_Amazon-SNS_Event-Driven-SNS-Compute%402x.03cb54865e1c586c26ee73f9dff0dc079125e9dc.png)

### SNS - Overview

- **event producer**는 오직 하나의 SNS topic에만 메시지를 보냄
- **event receiver**(구독)는 원하는 만큼 많이 둘 수 있음
- 해당 topic을 구독하는 각 subscriber(= receiver)는 모든 메시지를 수신함 (메시지 필터 기능을 쓰지 않는다면)
- 각 topic 별로 최대 12,500,000 구독을 할 수 있음
- 최대 100,000 topic

### SNS - SNS integrates with a lot of AWS services

- 여러 AWS 서비스들이 알림(notification)을 위해 직접 SNS에 데이터를 전송할 수 있음

### SNS - How to publish

- Topic Publish (SDK 사용)
  - 토픽 생성
  - (하나 혹은 여러 개의) 구독 생성
  - 토픽을 구독

- Direct Publish (모바일 앱 SDK 사용)
  - 플랫폼 애플리케이션 생성
  - 플랫폼 엔드포인트 생성
  - 플랫폼 엔드포인트를 구독
  - Google GCM, Apple APNS, Amazon ADM 등과 호환

### SNS - Security

- **Encryption**:
  - HTTPS API를 통한 In-flight encryption
  - KMS 키를 통한 At-rest encryption
  - 클라이언트가 자체적으로 암호화/복호화를 원하는 경우 클라이언트 측 암호화 가능

- **Access Controls**: SNS API로의 액세스를 조정하는 IAM 정책
- **SNS Access Policies** (S3 버킷 정책과 유사)
  - 서로 다른 계정 간의 SNS 토픽에 액세스해야 하는 경우 유용함
  - 다른 서비스(SNS, S3...)가 SQS 토픽에 작성해야 하는 경우 유용함

## SNS + SQS

### Fan Out

![Fan Out](https://miro.medium.com/v2/resize:fit:723/1*DRrTtdyah9NHwR0VCm6MWA.png)

- SNS에 한번 푸쉬하고, 모든 SQS 큐를 구독자로 두어 이를 수신하도록 하는 방법
- 완전히 디커플링할 수 있음 / 데이터 손실 없음
- SQS는 다음과 같은 것들을 처리: Data Persistence(데이터 지속성), 지연 시간이 존재하는 처리 & 동일 작업의 수행
- 시간이 지남에 따라 더 많은 SQS 구독자를 추가할 수 있음
- SQS 큐의 **Access Policy**가 SNS에게 쓰기 권한을 허용하는지 확인할 것
- Cross-Region Delivery: SQS 큐는 다른 리전에서도 동작함

### Fan Out - Application: S3 Events to multiple queues

- **event type(e.g., object create)** 과 **prefix(e.g., images/)**
  - 오직 하나의 **S3 Event rule**만 가질 수 있음
- 만약 여러 개의 SQS 큐에 동일한 S3 이벤트를 전달하고자 한다면, fan-out을 사용
  - SQS 큐 뿐만 아니라, Lambda로도 가능

![s3 fan out](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/05/23/fanout-S3-simple-usecase-diagram.png)

### Application: SNS to Amazon S3 through Kinesis Data Firehose

- SNS는 Kinesis로도 전송할 수 있으며, 이를 통해 아래와 같은 솔루션 아키텍처를 가질 수도 있음

![S3 through Kinesis Data Firehose](https://docs.aws.amazon.com/images/sns/latest/dg/images/firehose-architecture-s3.png)

### SNS - FIFO Topic

![SNS FIFO](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2020/07/07/pub_sub_messaging.png)

- FIFO = First In First Out (토픽 내 메시지의 순서)
- SQS FIFO와 유사한 기능
  - 메시지 그룹 ID 기반으로 **정렬** (동일한 그룹 내 모든 메시지가 정렬됨)
  - Deduplication ID 또는 Content Based Deduplication를 통한 **중복제거**
- **오직 SQS FIFO 큐만을 구독자로 둘 수 있음**
- 제한된 처리량 (SQS FIFO와 동일한 처리량)

### SNS - SNS FIFO + SQS FIFO: Fan Out

![FIFO Fan Out](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2020/07/07/sns_fifo_two_subscriptions-1024x422.png)

- Fan Out + 정렬 + 중복 제거가 모두 필요한 상황에서 사용

### SNS - Message Filtering

![SNS message filtering](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/21/Payload-filtering-example.png)

- SNS 토픽의 구독 대상들에게 전달되는 메시지를 필터링 하는 데에 사용하는 JSON 정책
- 별도로 필터 정책이 없는 경우, 모든 메시지를 받게 됨

## Kinesis

### Kinesis - Overview

- 실시간으로 데이터를 **수집, 처리, 분석**하기 쉽게 만들어 줌
- 실시간으로 다음과 같은 데이터들을 수집:
  - 애플리케이션 로그
  - 통계, 지표
  - 웹사이트 클릭스트림
  - IoT 원격 측정 데이터
- **Kinesis Data Streams**: 데이터 스트림을 캡처, 처리, 보관
- **Kinesis Data Firehose**: 데이터 스트림을 AWS 데이터 스토어로 불러옴
- **Kinesis Data Analytics**: SQL 또는 Apache Flink로 데이터 스트림 분석
- **Kinesis Video Streams**: 비디오 스트림을 캡처, 처리, 보관

### Kinesis Data Streams

![Kinesis Data Streams](https://docs.aws.amazon.com/images/streams/latest/dev/images/architecture.png)

- 1일 ~ 365일 동안 보관
- 데이터를 재처리(재실행)할 수 있음
- 일단 데이터가 Kinesis로 삽입되고 나면, 삭제될 수 없음 (immutability)
- 동일한 파티션을 공유하는 데이터는 동일한 샤드로 이동 (ordering)
- Producers: AWS SDK, Kinesis Producer Library (KPL), Kinesis Agent
- Consumers:
  - 직접 작성: Kinesis Client Library (KCL), AWS SDK
  - 관리형: AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics

### Kinesis Data Streams - Capacity Modes

- **Provisioned mode**:
  - 프로비저닝 및 스케일링될 샤드의 개수를 직접 또는 API를 통해 선택
  - 각 샤드는 입력에 1MB/s (또는 매초 1000 레코드)
  - 갹 사드는 출력에 2MB/s (classic 또는 enhanced fan-out consumer의 경우)
  - 매 시간 프로비전된 샤드 마다 비용 지불
- **On-demand mode**:
  - 프로비저닝이나 용량을 관리할 필요가 없음
  - 기본 용량으로 프로비전됨 (4MB/s 또는 매초 4000 레코드 입력)
  - 지난 30일 동안 관측한 최대 처리량에 기반하여 자동 스케일링
  - 시간 당 스트림 & GB 당 데이터 in/out에 따라 비용 지불

### Kinesis Data Streams - Security

- IAM 정책을 통한 Control Access / Authorization
- HTTPS 엔드포인트를 통한 in-flight encryption
- KMS를 통한 at-rest encryption
- 클라이언트 측에서 데이터 암호화/복호화 가능 (더 어렵긴 함)
- Kinesis가 VPC에 액세스할 수 있도록 하는 VPC Endpoint
- CloudTrail을 통한 모니터링 API 호출

### Kinesis Data Firehose

![Kinesis Data Firehose](https://d1.awsstatic.com/pdp-how-it-works-assets/product-page-diagram_Amazon-KDF_HIW-V2-Updated-Diagram@2x.6e531854393eabf782f5a6d6d3b63f2e74de0db4.png)

### Kinesis Data Firehose - Overview

- 완전 관리형 서비스, 관리할 필요 없음, 자동 스케일링, 서버리스
  - AWS: Redshift / S3 / OpenSearch
  - 3rd party partner: Splunk / MongoDB / DataDog / NewRelic / ...
  - Custom: 어떤 HTTP 엔드포인트로든 전송 가능
- Firehose를 통해 전송된 데이터 당 비용 지불
- **거의 실시간 (= 실시간 아님)**
  - 전체 배치(full batch)가 아닌 경우 최소 60초의 레이턴시 가짐
  - 또는 한번에 최소 1MB가 넘는 경우 약간 기다려야 함
- 여러 데이터 포맷, conversion, transformation, compression을 지원
- **AWS 람다를 통해 커스텀 데이터 변환 가능**
- **백업 S3 버킷으로 failed 또는 모든 데이터를 전송 가능**

### Kinesis Data Stream vs Firehose

- Kinesis Data Streams
  - 대규모 수집을 위한 스트리밍 서비스
  - 커스텀 코드 작성 (producer / consumer)
  - 실시간 (~200ms)
  - 스케일링 관리 (샤드 스플리팅 / 머징 ~ merging)
  - 1일 ~ 365일 동안 보관되는 데이터 스토리지
  - 리플레이 기능 지원
- Kinesis Data Firehose
  - S3 / Redshift / OpenSearch / 써드파티 / 커스텀 HTTP로 스트리밍 데이터 로드
  - 완전 관리형
  - 거의 실시간 (버퍼 시간 최소 60초)
  - 자동 스케일링
  - 데이터 스토리지 없음
  - 리플레이 기능 지원하지 않음

## Ordering Data

### Ordering Data - Kinesis

- 만약 AWS에 길 위 100대 트럭(`truck_1`, `truck_2`, ...)의 GPS 위치를 주기적으로 전송해야 한다고 생각해보자
- 여기서 트럭의 위치를 추적하기 위해, 각 트럭마다 순서대로 데이터를 consume하고자 하고자 한다
- 어떻게 Kinesis에 데이터를 전송해야 할까?
  - 답은 **`truck_id`를 "Partion Key" 값으로 사용하는 것**이다.
  - **똑같은 key는 항상 똑같은 shard에 저장되기 때문**

### Ordering Data - into SQS

- SQS standard에서는 정렬이 없음
- SQS FIFO의 경우, Group ID를 사용하지 않는다면 메시지는 도착한 순서대로 consume되며, **이는 오직 하나의 consumer에 의해 이루어짐**

![SQS FIFO Ordering 1](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/05/01/sqs_fifo_blog_img3-1024x256.png)
![SQS FIFO Ordering 2](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/05/02/sqs_fifo_blog_img4-1024x256.png)

- consumer의 개수를 늘리면서, 또 메시지가 서로 관련되어 있을 때 그룹화 하는 것도 원한다면
  - **Group ID**를 사용해야 함 (Kinesis에서의 Partition Key와 유사함)

![SQS FIFO using message group](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/05/02/sqs_fifo_blog_img6-1-1024x341.png)

### Ordering Data - Kinesis vs SQS Ordering

- **100개의 트럭, 5 Kinesis shards, 1 SQS FIFO가 있다고 가정**
- ***Kinesis Data Streams**:
  - 평균적으로 각 샤드 당 20개의 트럭 데이터
  - 각 샤드 내에서 데이터를 정렬하게 됨
  - 5개의 샤드가 있으므로, 병렬로 처리할 수 있는 최대 consumer 수는 5개
    - 이에 따라, 최대 5MB/s를 수신할 수 있음
- **SQS FIFO**
  - 오직 하나의 SQS FIFO 큐
  - 100개의 Group ID
  - 최대 100개의 consumer 가질 수 있음 (Group ID가 100개이기 때문)
  - 매초 최대 300개의 메시지를 가질 수 있음 (또는, 배치를 사용할 경우 3000)

## SQS vs SNS vs Kinesis

- **SQS**
  - consumer가 데이터를 pull해야함
  - 데이터는 consume되고 나면 삭제되어야 함
  - 원하는 만큼 많은 워커(= consumer)들을 가질 수 있음
  - 처리량을 프로비전 할 필요 없음
  - 오직 FIFO 큐를 통해서 정렬을 보장할 수 있음
  - 개별 메시지 딜레이 기능

- **SNS**
  - 여러 subscriber들에게 데이터를 push
  - 최대 12,500,000 subscriber
  - 데이터는 지속(persist)되지 않음 (전송되지 않으면 사라짐)
  - Pub/Sub (게시/구독)
  - 최대 100,000 topic
  - 처리량을 프로비전 할 필요 없음
  - SQS와 함께 사용하여 fan-out 아키텍처 패턴을 구성할 수 있음
  - SQS FIFO를 위한 FIFO 기능

- **Kinesis**
  - Standard: 데이터 pull
    - 샤드마다 2MB
  - Enhanced-fan out: 데이터 push
    - consumer별 샤드마다 2MB
  - 데이터 리플레이 가능
  - 실시간 빅데이터 분석과 ETL(Extract, Transform, Load) 목적
  - 샤드 레벨에서 정렬됨
  - X일 이후 데이터가 만료됨
  - Provisioned mode 또는 On-demand capacity mode 선택

## Amazon MQ

- SQS, SNS는 클라우드 네이티브 서비스: AWS 독점(proprietary) 프로토콜
- 온-프레미스를 통해 동작하는 기존 애플리케이션들은 다음과 같은 프로토콜을 사용했을 것임
  - MQTT, AMQP, STOMP, Openwire, WSS
- **클라우드로 마이그레이팅 할 때**, 애플리케이션을 SQS, SNS로 아예 다시 엔지니어링 하는 대신, Amazon MQ를 사용할 수 있음
- **Amazon MQ는 RabbitMQ, ActiveMQ에 대한 관리형 메시지 브로커 서비스**
- Amazon MQ은 SQS/SNS만큼 스케일링 할 수 없음
- Amazon MQ는 서버 위에서 동작하며, 장애 대처를 위해 Multi-AZ로 실행할 수 있음
- Amazon MQ queue 기능(~SQS)과, topic 기능(~SNS)를 모두 갖추고 있음

### Amazon MQ - High Availability

![MQ High Availability](https://docs.aws.amazon.com/images/amazon-mq/latest/developer-guide/images/amazon-mq-activemq-broker-architecture-active-standby.png)
