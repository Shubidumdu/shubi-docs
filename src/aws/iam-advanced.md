# IAM Advanced

## AWS Organizations

- 글로벌 서비스
- 여러 AWS 계정을 관리할 수 있도록 해줌
  - 메인 계정 = 관리 계정 (Management account)
  - 그 외의 계정 = 멤버 계정 (Member account)
- 멤버 계정들은 한 organization의 일부가 될 수 있음
- 모든 계정 간에 청구서를 통합(consolidate)할 수 있음 - single payment method
- 통합 이용을 통한 비용적 이득 (EC2, S3에 대한 볼륨 할인)
- **계정 간에 Reserved Instances나 Savings Plans 할인을 공유할 수 있음**
- AWS 계정 생성을 자동화하기 위해 API를 활용할 수 있음

### AWS Organizations - Organizational Units (OU) ~ Examples

![OU Example](https://towardsthecloud.com/wp-content/uploads/aws-organizations-structure-1024x933.webp)

### AWS Organizations - Pros

- **Advantages**
  - 다중 계정 vs 한 계정 내 여러 VPC ~ 계정을 여러개 둠으로써 보안 측면에서 더 나음
  - 비용 청구 목적으로 태깅(tagging) 표준을 둘 수 있음
  - 모든 계정에 대해 CloudTrail을 활성화하고, 중앙 S3 계정에 로그들을 보낼 수 있음
  - 중앙 로깅 계정에 CloudWatch 로그를 전송
  - 관리 목적으로 Cross Account Role을 구축할 수 있음
- **Security: Service Control Policies (SCP)**
  - User 또는 Role 역할을 제한하기 위해 IAM 정책을 OU 또는 계정 단위로 적용할 수 있음
  - SCP는 관리 계정에 적용할 수 없음 (full admin power)
  - 반드시 explicit allow를 가져야 함 (기본적으로는 아무것도 허용하지 않음 - IAM처럼)

### AWS Organiazations - SCP Examples ~ Blocklist and Allowlist strategies

- 기본적으로 모든 액션을 허용하고(ex. `AllowsAllActions`), 일부 권한에 대해 Deny 시키는 방법을 쓰거나 ~ Blocklist
- 기본적으로 모든 액션을 막아두고(별도로 권한 부여하지 않음), 일부 권한을 Allow 시키는 방법을 쓸 수 있음 ~ Allowlist

## IAM Conditions

- **aws:SourceIp**: API 호출이 이루어지는 클라이언트 IP를 제한

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": ["192.0.2.0/24"]
        }
      }
    }
  ]
}
```

- **aws:RequestedRegion**: API 호출이 이루어지는 지역을 제한

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["ec2:*", "rds:*", "dynamodb:*"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["eu-central-1", "eu-west-1"]
        }
      }
    }
  ]
}
```

- **ec2:ResourceTag**: 태그를 기반으로 제한

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:StartInstances", "ec2:StopInstances"],
      "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Project": "DataAnalytics",
          "aws:PrincipalTag/Department": "Data"
        }
      }
    }
  ]
}
```

- **aws:MultiFactorAuthPresent**: MFA를 강제

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
    },
    {
      "Effect": "Deny",
      "Action": ["ec2:StartInstances", "ec2:TerminateInstances"],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": false
        }
      }
    }
  ]
}
```

## IAM for S3

- **s3:ListBucket** 권한은 **arn:aws:s3:::test**에 적용됨
  - 버킷 레벨의 권한
- **s3:GetObject**, **s3:PutObject**, **s3:DeleteObject**는 <b>arn:awn:s3:::test\/\*</b>에 적용됨
  - 오브젝트 레벨의 권한

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::test",
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::test/*",
    }
  ]
}
```

## Resource Policies & aws:PrincipalOrgID

- **aws:PrincipalOrgID**는 특정 AWS Organization 멤버 계정의 액세스 권한을 제한하기 위하여 리소스 정책 내에 사용됨

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::2022-financial-data/*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": ["o-yyyyyyyyyy"]
        }
      }
    }
  ]
}
```

## IAM Roles vs. Resource Based Policies

- Cross account의 경우:
  - 하나의 리소스에 resource-based policy를 부여할 수 있음 (ex. S3 bucket policy)
  - 또는 IAM Role을 proxy로 사용할 수 있음

- **Role(이용자, 애플리케이션 또는 서비스)을 가정할 때는, 기존의 권한을 포기하고 role에 할당된 권한을 가져오게 됨**
- Resource-based policy를 사용할 때는, 기존의 권한을 포기하지 않아도 됨
  - **사례**:
    - accountA에 속한 이용자가 accountA에 있는 DynamoDB 테이블을 스캔하여 accountB에 있는 S3 버킷에 덤프해야 할 때는 Resource-based policy를 사용해야 함
  - 지원 대상: Amazon S3 버킷, SNS 토픽, SQS 큐 등등...

### IAM Roles vs. Resource Based Policies - Amazon EventBridge ~ Security

- rule이 동작할 때는, 타겟에 대한 권한이 필요하며 이는 다음의 두가지 형태가 될 수 있음
  - Resource-based policy: Lambda, SNS, SQS, CloudWatch Logs, API Gateway...
  - IAM Role: Kinesis stream, Systems Manager Run Command, ECS task...

## IAM Permission Boundaries

- IAM Permission Boundaries는 User 또는 Role에 의해 지원됨 (Group은 아님)
- 이는 IAM 엔티티가 가질 수 있는 최대 권한을 설정하도록 managed policy를 사용할 수 있게 해주는 고급 기능
- 예시: 아래처럼 S3, CloudWatch, EC2에 대한 권한을 허용하는 IAM Permission Boundary를 설정해둔 상태에서, 추가로 아래의 `iam:CreateUser`를 허용하도록 하더라도, 이는 효과가 없음 (추가 권한을 가질 수 없음)

```json
// IAM Permission Boundary
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:*", "cloudwatch:*", "ec2:*"],
      "Resource": "*"
    }
  ]
}
```

```json
// IAM Permissions Through IAM Policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:CreateUser",
      "Resource": "*"
    }
  ]
}
```

![Permission Boundaries Example](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/EffectivePermissions-rbp-boundary-id.png)

### IAM Permission Boundaries - Use Cases

- permission boundary 내에 있는 비관리자(non-admin)에게 권한을 위임 (ex. 새 IAM user 생성)
- 모든 개발자들에게 직접 정책을 할당하도록 허용하여 권한을 관리할 수 있도록 하면서도, 권한을 에스컬레이션(escalate)할 수는 없도록 막을 수 있음 (ex. 스스로 어드민을 부여해버린다던가)
- 한 특정 이용자에게 제한을 걸기에 유용함 (Organizations & SCP를 사용하여 전체 계정을 다루는 대신)

### IAM Policy Evaluation Logic

![IAM Policy Evaluation Logic](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/PolicyEvaluationHorizontal111621.png)

### Example IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sqs:*",
      "Effect": "Deny",
      "Resource": "*"
    },
    {
      "Action": [
        "sqs:DeleteQueue"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

- `sqs:CreateQueue`를 수행할 수 있는가?
  - **아니오**
- `sqs:DeleteQueue`를 수행할 수 있는가?
  - **아니오**
- `ec2:DescribeInstances`를 수행할 수 있는가?
  - **아니오**

## AWS IAM Identity Center (Successor to AWS Single Sign-On)

- 아래 모든 것들을 하나의 로그인(single sign-on)으로 통일할 수 있음
  - **AWS Organization 내에 있는 AWS 계정**
  - 비즈니스 클라우드 애플리케이션 (e.g., Salesforce, Box, Microsoft 365, ...)
  - SAML2.0 사용 애플리케이션
  - EC2 Windows Instances
- Identity Providers (ID 공급자)
  - IAM Identity Center 내 빌트인 Identity Store
  - 써드파티: Active Directory(AD), OneLogin, Okta...

### AWS IAM Identity Center - Fine-grained Permissions and Assignments

- **Multi-Account Permissions**
  - AWS Organization 내에 있는 여러 AWS 계정 간의 액세스 관리
  - Permission Sets - 이용자 및 그룹의 AWS 액세스 권한을 정의하기 위한 하나 이상의 IAM Policy 컬렉션

- **Application Assignments**
  - 여러 SAML 2.0 비즈니스 애플리케이션에 대한 SSO 액세스 (Salesforce, Box, Microsoft 365...)
  - URL, 인증서, 메타데이터 입력이 필수적임

- **Attribute-Based Access Control (ABAC)**
  - IAM Identity Center Identity Store에 저장된 이용자의 속성(attribute)에 기반한 Fine-grained permission
  - 예시: cost center, title, locale, ...
  - 사례: 일단 한번만 권한을 정의한 다음에는 속성이 변경될 때마다 AWS 액세스가 수정되도록 할 수 있음

## AWS Directory Services

### What is Microsoft Active Directory (AD)?

- AD Domain Services가 있는 모든 Windows Server에서 볼 수 있음
- **objects**에 대한 데이터베이스: 이용가 계정, 컴퓨터, 프린터, 파일 공유, 보안 그룹
- 보안 관리, 계정 생성, 권한 할당을 중앙에서 관리
- Object들은 **tree**안에 구성됨
- 여러 tree의 그룹은 **forest**가 됨

### AWS Directory Services - Types

- **AWS Managed Microsoft AD**
  - AWS 내에 자체적인 AD를 생성, 로컬에서 이용자들을 관리, MFA 지원
  - 온-프레미스 AD와 "신뢰 가능한" 연결을 구성할 수 있음
- **AD Connector**
  - 온-프레미스 AD로 리다이렉트하는 Directory Gateway (proxy)로, MFA 지원
  - 온-프레미스 AD에서 이용자들을 관리
- **Simple AD**
  - AWS 내 AD와 호환되는 관리형 디렉토리
  - 온-프레미스 AD와는 합칠 수 없음

### AWS Identity Center - Active Directory Setup

- **AWS Managed Microsoft AD (Directory Service)에 연결**
  - 즉시 사용 가능한 통합
- **직접 관리하는 Directory에 연결**
  - AWS Managed Microsoft AD를 사용해 Two-way Trust Relationship을 구축
  - 또는 AD Connector를 생성하여 프록시 구성

## AWS Control Tower

- 베스트 프랙티스에 맞게 **multi-account AWS 환경에서 안전하고 규정을 준수하도록 설정 및 관리**를 할 수 있도록 해주는 쉬운 방법
- AWS Control Tower는 계정 생성을 위해 AWS Organization을 사용

- 이점:
  - 몇 번의 클릭으로 환경을 구축할 수 있도록 자동화
  - guardrail을 통해 사용 중인 정책 관리를 자동화
  - 정책 위반을 감지하고 이를 수정
  - 인터랙티브 대시보드를 통해 규정 준수 내용을 모니터링

### AWS Control Tower - Guardrails

- 내 ControlTower 환경에 현재 진행중인 거버넌스(governance)를 전달 (모든 AWS Accounts에)
  - **Preventive Guardrail** - **SCP(Service Control Policies)를 사용** (e.g., 모든 계정 내 리전을 제한)
  - **Detective Guardrail** - **AWS Config을 사용** (e.g., 태그가 되지 않은 리소스들을 파악)
