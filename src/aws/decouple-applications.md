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
