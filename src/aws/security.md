# Security & Encryption

## Why encryption?

### **Encryption in flight (SSL)**

- 데이터가 전송되기 전에 암호화되고, 받은 이후에 복호화
- SSL 인증서가 암호화를 도움 (HTTPS)
- Encryption in flight는 MITM (man in the middle attack)이 일어나지 않음을 보장해줌
  - MITM? - 공격자가 이용자의 인터넷 서버와 트래픽의 목적지 사이에 끼어들어 전송을 가로채는 공격

### **Server side encryption at rest**

- 서버가 데이터를 전송받은 이후 이를 암호화
- 서버측에서 데이터를 전송하기 전에 복호화
- 하나의 키(보통 data key)를 통해 암호화된 형태로 저장
- 암호화/복호화 키는 반드시 어딘가에서 관리되는 오브젝트여야 하며, 서버가 여기에 반드시 접근 할 수 있어야 함

### **Client side encryption**

- 데이터가 클라이언트를 통해 암호화되고, 서버 측에서는 이를 복호화할 일이 없음
- 복호화 역시 데이터를 넘겨받은 클라이언트 측에서 이루어짐
- 서버 쪽에서는 데이터를 복호화할 수 없어야 함
- Envelope Encryption을 활용할 수 있음

## AWS KMS (Key Management Service)

- AWS 서비스에서 "암호화"와 관련된 내용을 들었다면 대부분은 KMS와 관련된 것
- AWS가 암호화 키를 관리해 줌
- 권한 처리를 위해 IAM과 호환
- 데이터에 대한 액세스를 통제하는 쉬운 방법
- CloudTrail을 통해 KMS 키 사용을 감시할 수 있음
- 대부분의 AWS 서비스에서 원활하게 호환됨 (EBS, S3, RDS, SSM...)
- **절대로 secret을 plaintext로 저장하지 말 것, 특히 코드에서!**
  - KMS 키 암호화는 API 호출을 통해서도 가능 (SDK, CLI)
  - 암호화된 secret은 코드 또는 환경 변수에 저장될 수 있음

### KMS Keys Types

- **KMS Keys는 KMS Customer Master Key의 새로운 이름**
- **대칭키 (AES-256 Keys)**
  - 하나의 키로 암호화/복호화 가능
  - KMS를 이용하는 AWS 서비스는 대칭 CMKs(Customer Managed Keys)를 이용함
  - 절대로 복호화된 KMS 키에 액세스할 수는 없음 (사용하려면 반드시 KMS API를 호출해야 함)

- **비대칭키 (RSA & ECC key pairs)**
  - Public Key(암호화)와 Private Key(복호화)가 한쌍
  - 암호화/복호화에 사용하거나, 서명/확인 작업에 사용
  - Public Key는 다운로드 가능하지만, 복호화된 Private Key에는 액세스할 수 없음
  - 사례: KMS API를 호출할 수 없는 AWS 외부 이용자를 통해 이루어진 암호화

- KMS 키의 종류
  - AWS Owned Key (무료): SS3-S3, SSE-SQS, SSE-DDB (기본 키)
  - AWS Managed Key: **무료** (aws/service-name ~ ex. aws/rds 또는 aws/ebs)
  - Customer managed keys created in KMS: **한달에 $1**
  - Customer managed keys imported (반드시 대칭키): **한달에 $1**
  - \+ KMS에 대한 API 호출에 따라 비용 지불 ($0.03 / 10000 호출당)

- Automatic Key rotation:
  - AWS-managed KMS Key: 매년마다 자동으로 이루어짐
  - Customer-managed KMS Key: (반드시 활성화됨) 매년마다 자동으로 이루어짐
  - Imported KMS Key: alias를 통해 가능할 때마다 수동으로만 로테이션

### Copying Snapshots across regions

![Copying Snapshots across regions with KMS](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/volume-reencrypt.png)

### KMS Key Policies

- KMS 키에 대한 액세스를 통제, S3로 치자면 bucket policies와 유사함
- 차이점: Key Policy 없이는 액세스를 관리할 수 없음

- **Default KMS Key Policy**:
  - 구체적인 KMS Key Policy를 전달하지 않았을 때 생성
  - 루트 User(전체 AWS 계정)에게 키에 대한 완전한 액세스 권한을 부여
- **Custom KMS Key Policy**:
  - KMS Key에 액세스할 수 있는 User, Role을 정의
  - 키를 관리할 수 있는 대상을 정의
  - 내 KMS 키에 대해 계정 간 액세스를 구현하는 데에 유용

### Copying Snapshots across accounts

1. 스냅샷을 생성하여 직접 보유한 KMS 키(Customer Managed Key)로 암호화
2. **cross-account 액세스 권한을 허용하도록 KMS Key Policy를 부여** (하단 참조)
3. 암호화된 스냅샷을 공유
4. (타겟에서) 스냅샷의 복제본을 생성하고, 이를 계정 내 CMK(Customer Managed Key)로 암호화
5. 스냅샷으로부터 볼륨을 생성

```json
{
  "Sid": "Allow use of the key with destination account",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::TARGET-ACCOUNT-ID:role/ROLENAME"
  },
  "Action": [
    "kms:Decrypt",
    "kms:CreateGrant"
  ],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:ViaService": "ec2.REGION.amazonaws.com",
      "kms:CallerAccount": "TARGET-ACCOUNT-ID"
    }
  }
}
```

## KMS Multi-Region Keys

![KMS Multi-Region Keys](https://docs.aws.amazon.com/images/kms/latest/developerguide/images/multi-region-keys.png)

- 다른 AWS 리전에서도 상호 교환이 가능한 동일 KMS Key
- Multi-Region key는 동일한 key ID, key material, automatic rotation을 가짐
- 한 리전에서 암호화하고, 다른 리전에서 복호화할 수 있음
- 다시 암호화하거나, 크로스 리전 API 호출을 수행할 필요 없음
- KMS Multi-Region은 글로벌하지 않음 (Primary + Replicas)
- 각각의 Multi-Region Key는 **독립적으로** 관리됨
- **사례**: 글로벌 클라이언트 사이드 암호화, 글로벌 DynamoDB 또는 글로벌 Aurora 암호화

### DynamoDB Global Tables and KMS Multi-Region Keys - Client-Side encryption

- **Amazon DynamoDB Encryption Client**를 사용하는 DynamoDB 테이블 내에 클라이언트 측에서 특정 attribute만 암호화할 수 있음
- GlobalTables와 함께 사용되어, 클라이언트 측에서 암호화된 데이터는 다른 리전으로도 복제됨
- 만약 Multi-Region Key를 사용한다면, 이를 DynamoDB Global 테이블과 동일한 리전에 복제하여 해당 리전 내 클라이언트들이 클라이언트 복호화를 수행하기 위해 KMS API을 낮은 레이턴시로 호출할 수 있게됨
- 클라이언트 측 암호화를 통해 특정한 필드만 보호할 수 있고, 오직 API 키에 액세스 권한을 보유한 클라이언트만 복호화를 수행하도록 보장할 수 있음

![DynamoDB Global Tables and KMS Multi-Region Keys](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/06/15/Encrypt-global-data-ForSocialr.jpg)

### Global Aurora and KMS Multi-Region Keys - Client-Side encryption

- **AWS Encryption SDK**를 통해 Aurora 테이블 내 특정 attribute를 클라이언트 측 암호화 할 수 있음
- Aurora Global Table과 함께 사용하여, 클라이언트 측 암호화된 데이터를 다른 리전에 복제
- 만약 Multi-Region Key를 사용한다면, Global Aurora DB와 동일한 리전에 복제하여 해당 리전 내 클라이언트들이 클라이언트 복호화를 수행하기 위해 KMS API을 낮은 레이턴시로 호출할 수 있게됨
- 클라이언트 측 암호화를 통해 특정한 필드만 보호할 수 있고, 오직 API 키에 액세스 권한을 보유한 클라이언트만 복호화를 수행하도록 보장할 수 있으며, **심지어 DB 어드민으로부터도 특정 필드를 보호할 수 있음**

![Global Aurora and KMS Multi-Region Keys](https://velog.velcdn.com/images/jungmyeong96/post/ee17c8bd-5734-45e2-b0f6-a193becd75fb/image.png)

## S3 Replication Encryption Considerations

- **기본적으로 암호화되지 않은 오브젝트와 SSE-S3로 암호회된 오브젝트가 복제됨**
- SSE-C(Customer provided key)로 암호화된 오브젝트는 복제될 수 없음
- **SSE-KMS로 암호화된 오브젝트들의 경우**, 옵션을 활성화해야함
  - 타겟 버킷 내에 오브젝트를 암호화하기 위해 어떤 KMS 키를 쓸 것인지 명시
  - 타겟 키를 위한 KMS Key Policy를 적용
  - 소스 KMS Key에 대한 복호화를 위해 `kms:Decrypt`와 타겟 KMS Key에 대한 암호화를 위해 `kms:Encrypt`를 가진 IAM Role이 필요함
  - KMS 쓰로틀링 에러가 발생할 수 있으며, 이 경우 Service Quota의 증가를 요청할 수 있음
- **Multi-Region AWS KMS Keys를 사용할 수는 있으나, 아직까지는 Amazon S3에서 이를 independent key로 다루고 있음 (오브젝트들은 여전히 복호화/암호화 처리가 될 것임)**

## AMI Sharing Process Encrypted via KMS

1. AMI in Source Account는 Source Account로부터 KMS Key로 암호화됨
2. 이미지 속성을 수정하여, 타겟 AWS 계정에 **Launch Permission**을 반드시 추가
3. AMI 참조 스냅샷을 암호화하는 데 사용한 KMS Key를 타겟 계정 또는 IAM Role와 반드시 공유
4. IAM Role 또는 타겟 계정 내 User는 반드시 DescribeKey, ReEcrypted, CreateGrant, Decrypt에 대한 권한을 보유해야 함
5. AMI로부터 EC2 인스턴스를 실행할 때, 원한다면 타겟 계정은 본인 계정 내 새로운 KMS Key를 정의해서 볼륨을 새로 암호화할 수 있음

## SSM Parameter Store

- configuration 및 secrets를 위한 안전한 스토리지
- KMS를 통한 암호화 (선택 사항)
- serverless, scalable, durable, easy SDK
- configuration / secrets에 대한 버전 트래킹
- IAM을 통해 보안
- Amazon EventBridge를 통한 알림
- CloudFormation과 호환

### SSM Parameter Store Hierarchy

- /my-department/
  - my-app/
    - dev/ ~ Dev Lambda 함수 내 `GetParameters` 또는 `GetParametersByPath` API 사용
      - db-url
      - db-password
    - prod/ ~ Prod Lambda 함수 내 `GetParameters` 또는 `GetParametersByPath` API 사용
      - db-url
      - db-password
  - other-app/
- /other-department/
- /aws/reference/secretsmanager/secret_ID_in_Secrets_Manager
- /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 (public)

<table id="w810aac21c12c17b9c11">
  <thead>
    <tr>
        <th></th>
        <th>Standard</th>
        <th>Advanced</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>
            <p>Total number of parameters allowed</p>
            <p>(per AWS account and AWS Region)</p>
        </td>
        <td>
            <p>10,000</p>
        </td>
        <td>
            <p>100,000</p>
        </td>
    </tr>
    <tr>
        <td>
            <p>Maximum size of a parameter value</p>
        </td>
        <td>
            <p>4 KB</p>
        </td>
        <td>
            <p>8 KB</p>
        </td>
    </tr>
    <tr>
        <td>
            <p>Parameter policies available</p>
        </td>
        <td>
            <p>No</p>
        </td>
        <td>
            <p>Yes</p>
        </td>
    </tr>
    <tr>
        <td>
            <p>Cost</p>
        </td>
        <td>
            <p>No additional charge</p>
        </td>
        <td>
            <p>Charges apply</p>
        </td>
    </tr>
  </tbody>
</table>

### Parameters Policies (for advanced parameters)

- 파라미터에 TTL을 할당(만료일자)하여 패스워드와 같은 민감 데이터를 수정 또는 삭제하도록 강제할 수 있음
- 동시에 여러 개의 정책 할당 가능

- Expiration (to delete a parameter)

```json
{
  "Type": "Expiration",
  "Version": "1.0",
  "Attributes": {
    "Timestamp": "2020-12-02T21:34:33.000Z"
  }
}
```

- ExpirationNotification (EventBridge)

```json
{
  "Type": "ExpirationNotification",
  "Version": "1.0",
  "Attributes": {
    "Before": "15",
    "Unit": "Days"
  }
}
```

- NoChangeNotification (EventBridge)

```json
{
  "Type": "NoChangeNotification",
  "Version": "1.0",
  "Attributes": {
    "After": "20",
    "Unit": "Days"
  }
}
```

## AWS Secrets Manager

- secrets들을 저장하기 위한 새로운 서비스
- 매 X일 마다 **secret의 rotation**을 강제하는 기능
- rotation 작업에 있어 secret의 생성을 자동화 (Lambda 사용)
- **Amazon RDS**(MySQL, PostgreSQL, Aurora)와 호환
- KMS를 통해 secret 암호화
- 대부분은 RDS와 함께 사용

### AWS Secrets Manager - Multi-Region Secrets

- 여러 AWS 리전 간에 secret을 복제
- Secrets Manager는 primary Secret에 싱크를 맞춘 read replica를 보유
- read replica Secret을 standalone Secret으로 승격시킬 수 있음
- 사례: Multi-region 앱, 재해 복구 전략, multi-region DB...

![AWS Secrets Manager multi region secret](https://disaster-recovery.workshop.aws/images/secretmanager-architecture.png)

## AWS Certificate Manager (ACM)

- **TLS Certificates(인증서)**들을 쉽게 프로비저닝, 관리, 배포
- 웹사이트에서의 in-flight 암호화를 제공 (HTTPS)
- public/private TLS 인증서를 모두 지원
- public TLS 인증서의 경우 무료
- 자동으로 TLS 인증서 갱신
- 다음과 호환 (TLS 인증서 가져오기)
  - Elastic Load Balancers (CLB, ALB, NLB)
  - CloudFront Distributions
  - APIs on API Gateway
- EC2에는 ACM을 사용할 수 없음 (can't be extracted)

![ACM Example on ELB](https://i.stack.imgur.com/eNLr3.png)

### ACM - Requesting Public Certificates

1. 인증서에 도메인 네임들을 추가

   - Fully Qualified Domain Name (FQDN): `corp.example.com`
   - Wildcard Domain: `*.example.com`

2. Validation 방식 선택: DNS Validation 또는 Email Validation

   - DNS Validation이 자동화 목적으로 선호됨
   - Email Validation은 WHOIS DB 내에 있는 contact 주소로 이메일이 전송됨
   - DNS Validation은 DNS 설정 내 CNAME 레코드를 활용함 (ex. Route53)

3. 검증에 몇 시간 정도가 소요됨
4. Public 인증서는 자동 갱신에 등록됨

   - ACM에서 생성한 모든 인증서는 만료 60일 전에 자동으로 갱신됨

### ACM - Importing Public Certificates

- ACM 외부에서 인증서를 생성하고, 이를 import하여 사용할 수 있는 옵션
- **자동 갱신 불가**, 반드시 만료 전에 새 인증서를 import 해야함
- 만료 45일 전부터 **ACM이 매일 만료 이벤트를 전송함**
  - 시작 일자는 변경 가능
  - 해당 이벤트는 EventBridge에 나타남
- **AWS Config**은 인증서 만료를 체크하기 위해 `acm-certificate-expiration-check`란 이름으로 관리되는 rule을 가짐 (날짜는 설정 가능)

### API Gateway - Endpoint Types

- **Edge-Optimized (기본)**: 글로벌 클라이언트용
  - CloudFront Edge location을 통해 요청이 라우트됨 (레이턴시 향상)
  - API Gateway는 여전히 오직 하나의 리전에만 있음
- **Regional**:
  - 동일한 리전 내 클라이언트들을 위한 것
  - CloudFront와 매뉴얼하게 조합할 수 있음 (캐싱 전략 및 배포를 더 많이 통제할 수 있음)
- **Private**:
  - 인터페이스 VPC 엔드포인트(ENI)를 통해 내 VPC를 통해서만 액세스 할 수 있음
  - 액세스를 정의하기 위해 Resource Policy를 사용

### ACM - Integration with API Gateway

- API Gateway 내에 **Custom Domain Name**을 생성
- **Edge-Optimized** (기본): 글로벌 클라이언트용
  - CloudFront Edge location을 통해 요청이 라우트됨 (레이턴시 향상)
  - API Gateway는 여전히 오직 하나의 리전에만 있음
  - **TLS 인증서는 반드시 CloudFront와 동일한 리전에 있어야 함** (= `us-east-1`)
  - 이후 CNAME 또는 A-Alias 레코드를 설정 (A-Alias 쪽이 선호)
- **Regional**:
  - 동일한 리전 내 클라이언트들을 위한 것
  - **TLS 인증서는 반드시 API Stage와 동일한 리전 내 API Gateway에서 import되어야 함**
  - 이후 CNAME 또는 A-Alias 레코드를 설정 (A-Alias 쪽이 선호)

## AWS WAF - Web Application Firewall

- 일반적인 웹 취약점(exploit)으로부터 웹 애플리케이션을 보호 (layer 7)
- **layer 7은 HTTP** (vs layer 4는 TCP/UDP)
- 다음에 배포 가능
  - Application Load Balancer
  - API Gateway
  - CloudFront
  - AppSync GraphQL API
  - Cognito User Pool

### AWS WAF - Web ACL

- Web ACL (Web Access Control List) Rules:
  - **IP Set**: **최대 10,000 IP 주소** - 더 많은 IP를 할당하려 한다면 여러 Rule을 추가하면 됨
  - **SQL injection**과 **Cross-Site Scripting**(**XSS**)같은 일반적인 공격으로부터 HTTP 헤더/바디, 또는 URI 스트링을 보호
  - 크기 제한, **geo-match** (**block countries**)
  - **Rate-based rules** (이벤트 발생 횟수 카운팅) - **DDoS 보호**
- Web ACL은 CloudFront를 제외하고는 Regional하게 적용
- Rule Group = **Web ACL에 추가될 수 있는 Rule의 재사용 가능한 집합**

### AWS WAF - Fixed IP while using WAF with a Load Balancer

- WAF는 NLB(Network Load Balancer)를 지원하지 않음 (layer 4)
- ALB에 WAF를 활성화하려면 고정 IP(fixed IP)가 요구되며, 이를 위해 Global Accelerator가 필요

## AWS Shield: protect from DDoS attack

- **DDoS**: Distributed Denial of Service - 동시에 많은 요청을 보내는 것
- **AWS Shield Standard**:
  - 모든 AWS 고객들에 대해 무료로 활성화되는 서비스
  - SYN/UDP Floods, Reflection attacks 또는 그 외의 layer 3/layer 4 공격에 대한 보호
- **AWS Shield Advanced**:
  - 선택적인 DDoS 완화(mitigation) 서비스 (organization별 매달 $3,000)
  - *AC2, ELB, CloudFront, Global Accelerator, Route 53*에 대한 보다 정교한 공격에 대한 보호
  - AWS DDoS response team(DRP)과 항상 연결
  - DDoS로 인한 usage spike 동안에 발생하는 높은 비용으로부터 보호
  - Shield Advanced의 automatic application layer DDoS mitigation이 자동으로 AWS WAF Rule을 생성/평가/배포하여 layer 7 공격을 완화

## AWS Firewall Manager

- **AWS Organization 내 모든 계정에 대한 Rule을 관리**
- Security Policy: security rule에 대한 common set
  - WAF Rule (ALB, API Gateway, CloudFront)
  - AWS Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
  - Security Groups for EC2, ALB and ENI resources in VPC
  - AWS Network Firewall (VPC 레벨)
  - Amazon Route 53 Resolver DNS Firewall
  - 리전 레벨에서 생성된 Policy
- **내 Organization 내에 존재하거나 추후 추가될 모든 계정에서 생성되는 새 리소스들에 Rule이 적용됨(compliance에 유용함)**

## WAF vs. Firewall Manager vs. Shield

- **WAF, Sheild, Firewall Manager은 모두 보안을 위해 함께 사용됨**
- WAF에서는 Web ACL Rule을 정의함
- 리소스에 대한 세분화된 보호를 원한다면 WAF의 단독 사용이 적절한 선택임
- 만약, AWS WAF가 계정 간에 사용되어야 하거나, WAF 설정을 빠르게 하고 싶거나, 새 리소스에 대한 보호를 자동화하고 싶으면, AWS WAF에 Firewall Manager를 사용
- Shield Advanced는 AWS WAF에 추가로 Shield Response Team(SRT)로부터의 전담(dedicated) 지원과 고급 리포팅 같은 추가적인 기능을 제공함
- 반복적인 DDoS 공격에 취약한 상황이라면, Shield Advanced의 구매를 고려할 것

## AWS Best Practices for DDoS Resiliency

![AWS Best Practives for DDoS Resiliency](https://docs.aws.amazon.com/images/whitepapers/latest/aws-best-practices-ddos-resiliency/images/ddos-resilient-architecture.jpg)

- **Edge Location Mitigation (BP1, BP3)**
  - **BP1 - CloudFront**
    - 웹 애플리케이션이 엣지에서 전달됨
    - 일반적인 DDoS 공격으로부터 보호 (SYN floods, UDP reflections...)
  - **BP1 - Global Accelerator**
    - 엣지로부터 애플리케이션 액세스
    - DDoS 보호를 위한 Shield와 연동됨
    - 만약 CloudFront와 백엔드가 호환되지 않는 경우 유용함
  - **BP3 - Route 53**
    - 엣지에서 Domain Name Resolution
    - DNS에서 DDoS 보호 매커니즘이 적용되어 있음
- **Infrastructure layer defense (BP1, BP3, BP6)**
  - EC2를 높은 트래픽으로부터 보호
  - Global Accelerator, Route 53, CloudFront, Elastic Load Balancing을 포함
- **Amazon EC2 with Auto Scaling (BP7)**
  - flash crowd 또는 DDoS 공격을 포함한 갑작스러운 트래픽 급증에 따른 스케일링을 도와줌
- **Elastic Load Balancing (BP6)**
  - ELB가 급증하는 트래픽에 맞춰 스케일링을 수행하고, 이에 따라 트래픽을 여러 EC2 인스턴스로 분산시킴
- **Application Layer Defense**
  - **악성 웹 요청에 대한 감지 및 필터링(BP1, BP2)**
    - CloudFront는 정적 컨텐츠를 캐시하고 이를 엣지 로케이션에서 전달하도록 하여 백엔드를 보호
    - AWS WAF는 CloudFront와 ALB에 사용되어 요청 시그니처에 기반하여 요청을 필터링하거나 블록할 수 있음
    - WAF의 rate-based rule은 악성 행위자(bad actor)에 대한 IP를 자동으로 블록
    - CloudFront는 특정 지역을 블록할 수 있음
  - **Shield Advanced (BP1, BP2, BP6)**
    - Shield Advanced automatic application layer DDos mitigation은 layer 7 공격을 완화하기 위해 자동으로 AWS WAF Rule을 생성/평가/배포
- **Attack surface reduction**
  - **AWS 리소스 난독화(obfuscating) (BP1, BP4, BP6)**
    - CloudFront, API Gateway, Elastic Load Balancing을 사용하여 백엔드 리소스(Lambda 함수, EC2 인스턴스)를 감춤
  - **Security Groups and Network ACLs (BP5)**
    - Security Group과 NACL을 통해 서브넷이나 ENI레벨에서 특정 IP에서 일어나는 트래픽을 필터링
    - Elastic IP도 AWS Shield Advanced를 통해 보호됨
  - **API 엔드포인트를 보호 (BP4)**
    - EC2, Lambda를 다른 곳에 숨김
    - Edge-optimized mode 또는 CloudFront + regional mode (DDoS에 대한 더 많은 통제)
    - WAF + API Gateway: burst limit, 헤더 필터링, API key 사용을 강제

## Amazon GuardDuty

- AWS 계정 보호를 위해 인공지능으로 위협 감지
- 머신러닝 알고리즘, 이상징후 감지(anomaly detection), 써드파티 데이터 사용
- One Click 활성화 (30 days trial), 소프트웨어 설치 필요 없음
- Input Data들에는 다음이 포함:
  - **CloudTrail Events Logs** - 비정상적인 API 호출, 무단 배포
    - **CloudTrail Management Events** - VPC 서브넷 생성, trail 생성..
    - **CloudTrail S3 Data Events** - 오브젝트 get/list/delete...
  - **VPC Flow Logs** - 비정상적인 내부 트래픽, 비정상적인 IP 주소
  - **DNS Logs** - DNS 쿼리 내에서 인코딩된 데이터를 전송하는 손상된 EC2 인스턴스
  - **Kubernetes Audit Logs** - 의심스러운 활동 및 잠재적인 EKS 클러스터 손상
- 발견 시 알림이 전달될 수 있도록 **EventBridge Rule**을 설정 가능
  - EventBridge rule을 AWS Lambda 또는 SNS로 향하게 할 수 있음
- **CryptoCurrency 공격을 보호할 수 있음** (**이를 위한 전용 검색 보유**)

![Amazon GuardDuty](https://d1.awsstatic.com/Security/Amazon-GuardDuty/Amazon-GuardDuty_HIW.057a144483974cb73ab5f3f87a50c7c79f6521fb.png)

## Amazon Inspector

- **자동화된 보안 평가(automated security assessments)**
- **For EC2 Instances**
  - **AWS System Manager (SSM) 에이전트** 활용
  - **의도치 않은 네트워크 접근 가능성**에 대한 분석
  - 실행 중인 OS에 대한 **known vulnerabilities**을 파악
- **For Container Images push to Amazon ECR**
  - 푸시된 컨테이너 이미지에 대한 평가
- **For Lambda Functions**
  - 함수 코드와 패키지 의존성의 취약성을 파악
  - 배포된 함수에 대한 평가
- AWS Security Hub에 리포팅 & 호환 가능
- Amazon Event Bridge로 결과 전송

### What does Amazon Inspector evaluate?

- **중요**: **오직 EC2 인스턴스, 컨테이너 이미지 & Lambda 함수에 대해서만 적용 가능**
- 필요하다면 인프라에 대해 지속적인 스캐닝 가능
- 패키지 취약점 (EC2, ECR & Lambda) - database of CVE(Common Vulnerabilities and Exposures)
- 네트워크 도달 범위 ~ Network Reachability (EC2)
- 우선 순위에 따라 모든 취약점과 관련한 risk score를 제공

## Amazon Macie

- Amazon Macie는 완전 관리형 데이터 보안 및 데이터 프라이버시 서비스
  - **머신러닝과 패턴 매칭을 통해 AWS 내 민감한 데이터를 발견 및 보호**
- Macie는 **개인 식별 정보(PII ~ Personally Identifiable Information)과 같은 민감한 데이터를 파악하고 경고해줌**

![Amazon Macie Example](https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/05/25/bdb1118-archive-deletion-macie-1.png)
