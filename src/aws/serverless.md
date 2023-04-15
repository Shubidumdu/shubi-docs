
# Serverless

## What's serverless?

- 서버리스는 개발자들이 더 이상 서버를 관리할 필요가 없다는 새로운 패러다임
- 그냥 코드 / 함수를 배포
- 초기에는 Serverless == FaaS (Fucntion as a Service)
- 서버리스는 AWS Lambda로부터 시작되었으나, 이제는 그 외의 것들도 포함하기 시작함: 데이터베이스, 메시징, 스토리지 등등...
- **서버리스가 말그대로 서버가 없음을 의미하지는 않음**
  - 서버를 관리 / 프로비저닝할 필요가 없다는 의미

## Serverless in AWS

- AWS Lambda
- DynamoDB
- AWS Cognito
- AWS API Gateway
- Amazon S3
- AWS SNS & SQS
- AWS Kinesis Data Firehose
- Aurora Serverless
- Step Functions
- Fargate

## AWS Lambda

### AWS Lambda - Why AWS Lambda

- Amazon EC2
  - 클라우드에 있는 가상 서버
  - RAM / CPU에 의해 제한됨
  - 끊임없이 실행됨
  - 스케일링 -> 서버의 추가/제거를 중재
- Amazon Lambda
  - 가상 **함수** - 관리할 서버가 따로 없음
  - 시간에 제약 - **짧게 실행**
  - **온-디맨드**로 실행됨
  - **스케일링이 자동으로 이루어짐**

### AWS Lambda - Benefits of AWS Lambda

- 간단한 요금 책정
  - 각 요청과 컴퓨팅 시간을 기반으로 지불
  - 프리티어 - 1,000,000 AWS Lambda 요청 & 400,000 GB의 컴퓨팅 시간
- 전체 AWS 서비스들와 호환
- 여러 프로그래밍 언어와 호환
- AWS CloudWatch를 통해 쉽게 모니터링
- 각 함수 별로 리소스 할당을 쉽게 할 수 있음 (최대 10GB RAM)
- RAM을 상승시키면 CPU와 네트워크 성능도 향상됨

### AWS Lambda - Language support

- Node.js (JS)
- Python
- Java (Java 8 compatible)
- C# (.NET Core)
- Golang
- C# / Powershell
- Ruby
- Custom Runtime API (커뮤니티 지원, ex. Rust)
- Lambda Container Image
  - 컨테이너 이미지가 반드시 Lambda Runtime API를 구현해야 함
  - 그 외의 임의의 Docker 이미지를 실행하는 데에는 ECS / Fargate가 선호됨

### AWS Lambda - Integrations Main ones

- API Gateway
- Kinesis
- DynamoDB
- S3
- CloudFront
- CloudWatch Events EventBridge
- CloudWatch Logs
- SNS
- SQS
- Cognito

### AWS Lambda - Pricing Example

- 각 **호출**마다:
  - 첫 1,000,000개의 요청은 무료
  - 그 이후의 1,000,000 요청마다 $0.20 (각 요청마다 $0.0000002)
- 각 **이용시간**마다: (1ms 단위로)
  - 매 달 400,000GB-seconds의 컴퓨팅 시간에 대해선 무료
    - == 1GB RAM인 함수로 400,000초
    - == 128MB RAM인 함수로 3,200,000초
  - 그 이후의 사용량에 대해서는 600,000GB-seconds에 대해 $1.00
- 결론적으로, **매우 싸기** 때문에 **인기가 많음**

### AWS Lambda - Limits to Know ~ **per region**

- **Execution**:
  - 메모리 할당: 128MB ~ 10GB (1MB 단위)
  - 최대 실행 시간: 900s (15분)
  - 환경 변수 (4KB)
  - "함수 컨테이너(function container)"의 디스크 할당량 (in /tmp): 512MB ~ 10GB
  - 동시 실행: 1000 (증가 가능)
- **Deployment**
  - 람다 함수 배포 사이즈 (.zip으로 압축): 50MB
  - 비압축 배포 사이즈 (code + dependencies): 250MB
  - 시작 시 다른 파일들을 불러오고자 한다면 `/tmp` 디렉토리를 사용할 수 있음
  - 환경 변수 사이즈: 4KB

### AWS Lambda - Customization At The Edge

- 많은 최신 애플리케이션은 엣지에서 어떤 형태의 로직을 실행함
- **Edge Function**:
  - CloudFront 배포에 작성 및 연결된 코드
  - 레이턴시를 줄이기 위해 이용자에게 가까운 위치에서 실행됨
- CloudFront는 두 가지 타입의 방법을 제시
  - **CloudFront Functions**
  - **Lambda@Edge**
- 별도로 서버를 관리할 필요 없고, 글로벌로 배포됨
- 사례: CDN 컨텐츠를 커스터마이징
- 사용할 때에만 비용 지불
- 완전한 서버리스

### AWS Lambda - CloudFront Functions & Lambda@Edge ~ Use Cases

- 웹사이트 보안 & 프라이버시
- 엣지에 동적 웹 앱 배포
- SEO
- 오리진 및 데이터 센터 간에 지능적인 라우팅
- 엣지에서 봇 차단(bot mitigation)
- 실시간 이미지 변환
- A/B 테스팅
- 이용자 인증(authentication) 및 권한 부여(authorization)
- 이용자 우선순위 지정
- 이용자 트래킹 & 분석

### CloudFront Functions

- JS로 작성되는 가벼운 함수
- 지연 시간에 민감한, 대규모 CDN 커스터마이징에 적합
- ms 미만의 startup 시간, **초당 million(백만개) 요청**
- viewer 요청과 응답을 변경하는 데에 사용
  - **Viewer Request**: CloudFront가 viewer로부터 요청을 받은 후
  - **Viewer Response**: CloudFront가 viewer로 응답을 보내기 전

### Lambda@Edge

- NodeJS 또는 Python을 통해 작성되는 람다 함수
- **초당 1000개의 요청** 수준으로 스케일링 가능
- CloudFront 요청/응답을 변경하기 위해 사용:
  - **Viewer Request** - CloudFront가 viewer로부터 요청을 받은 이후
  - **Origin Request** - CloudFront가 origin으로 요청을 보내기 전
  - **Origin Response** - CloudFront가 origin으로부터 응답을 받은 이후
  - **Viewer Response** - CloudFront가 viewer로 응답을 보내기 전
- 함수를 AWS 리전(us-east-1)에서 생성한 다음, CloudFront가 해당 위치로 복제

### CloudFront Functions vs. Lambda@Edge

- **CloudFront Functions**
  - Cache key normalization
    - 요청 attributes들을 변환(헤더, 쿠키, 쿼리스트링, URL)하여 최적의 캐시 키를 생성
  - Header manipulation
    - 요청 또는 응답에서 HTTP 헤더를 삽입/수정/삭제
  - URL 재작성 또는 리다이렉트
  - 요청 인증 & 권한 부여
    - 요청 허용/거부를 위해 이용자 생성 토큰(ex. JWT)을 생성 및 검증
- **Lambda@Edge**
  - 더 긴 실행 시간 (몇 ms)
  - 조정 가능한 CPU 또는 메모리
  - 써드파티 라이브러리에 의존 (ex. 다른 AWS 서비스에 접근하기 위해 AWS SDK 사용)
  - 네트워크 액세스를 통해 처리에 필요한 외부 서비스를 사용
  - 파일 시스템 액세스 또는 HTTP 요청의 바디에 액세스

### AWS Lambda - by default

- 기본적으로, 람다 함수는 내가 보유한 VPC의 외부에서 실행됨 (AWS가 소유한 VPC에서)
- 그래서 내 VPC에 있는 리소스들에 액세스할 수 없음 (RDS, ElastiCache, internal ELB...)

### AWS Lambda - in VPC

- 반드시 VPC ID, 서브넷, Security Group을 정의해야 함
- Lambda가 내 서브넷 내에 ENI(Elastic Network Interface)를 생성함

### AWS Lambda - with RDS Proxy

- 만약 Lambda 함수가 데이터베이스에 직접 연결된다면, 부하가 높은 상태에서 너무 많은 DB open이 일어날 것임
- RDS Proxy
  - DB 연결을 풀링 및 공유함으로써 scalability 향상
  - failover 시간을 66% 감소시키고, 연결을 보존함으로써 availability 향상
  - IAM 인증을 적용하고 Secret Manager에 자격증명을 보관함으로써 security 향상
- **람다 함수는 반드시 VPC에 배포되어야 함** -> ***RDS Proxy는 절대로 public하게 접근가능해서는 안되기 때문**

### AWS Lambda - Invoking Lambda from RDS & Aurora

- DB 인스턴스 내에서 람다 함수를 호출
- DB 내에서 **데이터 이벤트**를 처리할 수 있음
- **RDS for PostgreSQL과 Aurora MySQL**을 지원
- DB 인스턴스에서 **Lambda 함수로 향하는 아웃바운드 트래픽을 반드시 허용해야 함** (Public, NAT GW, VPC Endpoints)
- **DB 인스턴스는 반드시 Lambda 함수를 실행하기 위해 요구되는 권한을 갖고 있어야 함** (Lambda Resource-based Policy & IAM Policy)

### RDS Event Notifications

- DB 인스턴스 자체에 대한 정보(생성, 중지, 시작, ..)를 알려주는 Notification
- 데이터 자체에 대한 정보는 얻을 수 없음
- 다음의 이벤트 카테고리들을 구독:
  - **DB instance, DB snapsho, DB Parameter Group, DB Security Group, RDS Proxy, Custom Engine Version**
- 거의 실시간 이벤트 (최대 5분)
- SNS로 알림을 전달하거나 EventBridge를 통해 이벤트를 구독

## Amazon DynamoDB

- multi AZ로 replication을 함으로써 고가용성(high availability)을 지닌 완전 관리형 DB
- NoSQL DB (Not RDB) - transaction 지원
- 많은 양의 워크로드에 대해서도 스케일링 가능, 분산형 DB
- 매초 1 million 요청, 1 trillion 행, 100TB 스토리지
- 빠르고, 일관적인 성능 (single-digit ms)
- 보안, 인증 및 관리를 위해 IAM 지원
- 낮은 비용, 자동 스케일링 기능
- 유지보수하거나 패칭(patch)할 필요 없음, 항상 available함
- Standard & Infrequent Access (IA) Table Class

### DynamoDB - Basics

- DynamoDB는 **Table**들로 구성
- 각 테이블은 **Primary Key**를 가짐 (생성 시점에 반드시 결정되어야 함)
- 각 테이블은 무한한 개수의 Item(= row)를 가질 수 있음
- 각 아이템은 **attribute**를 가짐 (추후에도 추가될 수 있음 - null일 수 있음)
- 한 아이템의 최대 크기는 **400KB**
- 다음의 데이터 타입들이 지원됨:
  - **Scalar Types** - String, Number, Binary, Boolean, Null
  - **Document Types** - List, Map
  - **Set Types** - String Set, Number Set, Binary Set
- **결국, DynamoDB에서는 빠르게 스키마를 발전시킬 수 있음**

![DynamoDB Table Example](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/09/10/dynamodb-partition-key-1.gif)

### DynamoDB - Read/Write Capacity Modes

- 테이블 가용량(capcity)을 어떻게 관리할 것인지를 설정 (read/write throughput)

- **Provisioned Mode (기본)**
  - 초당 read/write 개수를 정의
  - 미리 capacity를 계획해야 함
  - **프로비전된** Read Capacity Unit(RCU)와 Write Capacity Units(WC)만큼 비용 지불
  - RCU & WCU에 대한 **오토 스케일링** 모드를 추가할 수 있음
- **On-Demand Mode**
  - 워크로드에 따라 자동으로 R/W가 스케일링됨
  - 따로 capacity를 계획하지 않아도 됨
  - 사용한 만큼 더 많은 비용 지불
  - **예측이 불가능한** 워크로드,  **steep sudden spikes**에 유용함,

### DynamoDB - DynamoDB Accelerator (DAX)

- DynamoDB를 위한 fully-managed, highly available, seamless 인-메모리 캐시
- **캐싱을 통해 읽기 혼잡도를 해결하는데 도움**
- **캐시된 데이터에 대해 ms 단위 레이턴시**
- 별도의 애플리케이션 로직 수정을 요구하지 않음 (기존 DynamoDB API들과 호환됨)
- 캐시에 5분의 TTL (기본)

### DynamoDB - DynamoDB Accelerator (DAX) vs. ElastiCache

- Amazon ElasticCache
  - aggregatio 결과를 저장
- DynamoDB Accelerator (DAX)
  - 개별 오브젝트 캐시
  - 캐시를 쿼리 & 스캔

### DynamoDB - Stream Processing

- 테이블에서의 item 수정(create/update/delete)의 ordered stream
- 사례:
  - 실시간 변화에 대한 반응 (이용자들에게 welcome email 전송)
  - 실시간 사용 분석
  - 파생(derivative) 테이블에 insert
  - cross-region replication 수행
  - DynamoDB 테이블의 변경에 따른 AWS Lambda 실행
- **DynamoDB Streams**
  - 24시간 보존
  - 제한된 수의 consumer
  - AWS Lambda Trigger 또는 DynamoDB Stream Kinesis adapter를 통해 처리
- **Kinesis Data Streams (newer)**
  - 1년 보존
  - 많은 수의 consumer
  - AWS Lambda, Kinesis Data Analytics, Kinesis Data Firehose, AWS Glue Streaming ETL 등을 통해 처리

![DynamoDB Streams](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2021/05/06/DDB-Design-patterns-v1.3.jpg)

### DynamoDB - Global Tables

- 여러 리전 내에 DynamoDB 테이블을 **낮은 레이턴시**로 액세스 가능한 상태로 만들어 줌
- Active-Active replication
- 어떤 리전에서든 **READ**, **WRTIE**작업을 수행할 수 있음
- 전제조건(pre-requisite)으로, DynamoDB Stream이 활성화되어 있어야 함

![DynamoDB Global Table](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/12/20/Globeimage.png)

### DynamoDB - Time To Live (TTL)

- 만료(expiry) 타임스탬프가 지나면 아이템을 삭제
- 사례:
  - 오직 현재 item들만 보유함으로써 저장된 데이터 줄이기
  - 규제 의무(regulatory obligations) 준수
  - 웹 세션 핸들링

### DynamoDB - Backups for disaster recovery

- point-in-time recovery(PITR)를 통한 지속적인 백업
  - 지난 35일 동안 선택적으로 활성화
  - Point-in-time recovery -> 백업 기간 내라면 어느 시점이건 가능
  - recovery 작업은 새로운 테이블을 생성함
- On-demand backups
  - 명시적으로 삭제하기 전까지 장기 보존을 위해 전체 백업
  - 성능이나 레이턴시에 영향을 끼치지 않음
  - AWS Backup 내에서 설정하거나 관리할 수 있음 (리전 간 복사 가능)
  - recovery 작업은 새로운 테이블을 생성함

### DynamoDB - Inregration with Amazon S3

- **S3로 내보내기 (PITR 활성화 필수) ~ export**

  - 최근 35일 동안의 어느 시점이건 실행 가능
  - 테이블의 읽기 가용량에 영향을 끼치지 않음
  - DynamoDB의 상위에서 데이터 분석을 수행
  - 감사(audit)를 위해 스냅샷을 유지
  - DynamoDB로 다시 가져오기 전에 S3 데이터 위에 ETL(Extract, Load, Transfer)을 수행
  - Dynamo JSON 또는 ION 포맷으로 내보내기

- **S3로부터 가져오기 ~ import**
  - CSV, Dynamo JSON 또는 ION 포맷으로 임포트
  - 어떤 write capacity도 소비하지 않음
  - 새로운 테이블을 생성함
  - 임포트 에러 발생 시 CloudWatch 로그로 로깅됨

## AWS API Gateway

### API Gateway - Overview

- AWS Lambda + API Gateway: 관리가 필요한 인프라가 없음
- WebSocker 프로토콜 지원
- API 버저닝 (v1, v2, ...)
- 저마다 다른 환경(dev, test, prod..) 처리
- 보안 처리 - 인증 및 권한 부여
- API 키 생성, 요청 쓰로틀링 처리
- 빠른 API 정의를 위한 Swagger / Open API 가져오기
- 요청/응답 검증 및 변환
- SDK 및 API 사양(specification) 생성
- API 응답 캐시

### API Gateway - Integrations High Level

- **Lambda Function**
  - 람다 함수 실행
  - AWS 람다를 통한 REST API 백엔드를 노출(expose)시키는 쉬운 방법
- **HTTP**
  - 백엔드에 HTTP 엔드포인트를 노출
  - 사례: 온-프레미스에서의 내부 HTTP API, ALB...
  - 왜? - 속도 제한, 캐싱, 이용자 인증, API 키, 등등..
- **AWS Service**
  - API Gateway를 통한 어떤 AWS API든 노출
  - 사례: AWS Step Function workflow 실행, SQS에 메시지 전송
  - 왜? - 인증 추가, 공개 배포, 속도 제한...

### API Gateway - Endpoint Types

- **Edge-Optimized (기본)**:
  - 글로벌 클라이언트 대상
  - CloudFront 엣지 로케이션을 통해 요청이 라우트됨 (latency 상승)
  - API Gateway 자체는 여전히 하나의 리전에 존재함
- **Regional**:
  - 동일한 리전 내에 있는 클라이언트 대상
  - CloudFront와 매뉴얼하게 조합될 수 있음 (캐싱 전략 및 배포에 대해 더 많은 제어 가능)
- **Private**:
  - 인터페이스 VPC 엔드포인트(ENI)를 통해, 오직 내 VPC로부터만 액세스가 가능함
  - 액세스를 정의하는 리소스 정책(resource policy)을 사용

### API Gateway - Security

- **User Authentication** ~ 아래의 것들을 사용
  - IAM Roles (내부적인 애플리케이션에 유용)
  - Cognito (외부 이용자들을 식별 ~ ex. mobile users)
  - Custom Authorizer (직접 로직 작성)
- **Custom Domain Name HTTPS** ~ AWS Certificate Manager(ACM)과 함께 사용
  - Edge-Optimized 엔드포인트를 사용하는 경우, 반드시 인증서가 **us-east-1**에 있어야 함
  - Regional 엔드포인트를 사용하는 경우, 반드시 API Gateway가 위치한 리전에 인증서가 있어야 함
  - Route 53 내에 CNAME 또는 A 레코드가 반드시 설정되어야 함

## AWS Step Functions

![Step functions](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2021/05/14/03_architecture_diagram.png)

- 람다 함수를 오케스트레이션하기 위해 서버리스의 시각적인 워크플로를 구축
- **기능**: sequence, parallel, conditions, timeouts, error handling, ...
- EC2, ECS, 온-프레미스 서버, API 게이트웨이, SQS 큐 등과 호환
- human approval한 기능을 만들 수 있음
- **사례**: 주문 처리, 데이터 프로세싱, 웹 애플리케이션, 어떤 워크플로든지

## Amazon Cognito

- 웹/모바일 애플리케이션과 상호작용하는 이용자들을 식별하기 위함
- **Cognito User Pools**:
  - 애플리케이션 이용자들에 대한 로그인(sign in) 기능
  - API Gateway & ALB와 호환
- **Cognito Identity Pools (Federated Identity)**:
  - 이용자들에게 AWS credential을 부여하여 AWS 리소스들에 직접 액세스할 수 있도록 함
  - Cognito User pool을 ID 공급자로서 활용
- **Cognito vs IAM**: "hundreds of users", "mobile users", "authenticate with SAML"

### Cognito - Cognito User Pools(CUP) ~ User Features

- 웹/모바일 앱 이용자에 대한 서버리스 DB 생성
- 간단하게 로그인: Username (or email) / password
- 패스워드 리셋 기능
- 이메일 & 휴대폰 번호 인증
- MFA 인증
- Federated Identities: 페이스북 이용자, Google 이용자, SAML...

### Cognito - Cognito User Pools(CUP) ~ Integrations

- CUP는 **API Gateway**와 **Application Load Balancer**와 호환됨

### Cognito - Cognito Identity Pools (Federated Identities)

- **이용자가 임시 AWS 자격 증명을 얻을 수 있도록 이용자들의 ID를 가져옴**
- 이용자 소스는 Cognito User Pools나 그외의 써드 파티 로그인 등이 될 수 있음
- **이용자들은 직접 또는 API Gateway를 통해 AWS 서비스들에 액세스할 수 있음**
- IAM 정책은 Cognito 내에 정의된 credential에 적용됨
- 세밀한 컨트롤(fine grained control)을 위해 user_id에 기반하여 커스터마이징 할 수 있음
- 따로 정의해두지 않으면 인증 이용자, 게스트 이용자들에게는 **Default IAM role**이 적용됨

![Cognito Identity Pools](https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/images/scenario-cup-cib.png)

### Cognito - Cognito Identity Pools ~ Row Level Security in DynamoDB

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:DeleteItem",
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:Query",
                "dynamodb:UpdateItem"
            ],
            "Resource": ["arn:aws:dynamodb:*:*:table/MyTable"],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "dynamodb:LeadingKeys": ["${cognito-identity.amazonaws.com:sub}"] // <-- HERE
                }
            }
        }
    ]
}
```
