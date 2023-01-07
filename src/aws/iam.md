# IAM

- IAM = Identity and Access Management
- 글로벌 서비스에 해당함. (별도로 지역이 정의되어 있지 않다.)
- 기본적으로는 계정 생성 시에 **루트 계정(Root account)**이 생성된다.
  - 루트 계정은 AWS 리소스에 대한 모든 권한을 갖는다.
  - 하지만, 모든 권한을 갖는 만큼, 위험하기 때문에 루트 계정은 당장에 계정 세팅이 필요한 경우에만 사용하고, 그 이후에는 이용하거나 공유하지 말아야 한다.
- 루트 계정을 사용하는 대신, **유저(User)**를 만들어 주어야 하는데, 이는 한 조직 내 하나의 이용자를 가리키며, 그룹에 속할 수 있다.
- **그룹(Group)**은 오직 유저만 포함할 수 있으며, 다른 그룹은 포함할 수 없다.
- 유저가 꼭 하나의 그룹에 속해야 할 필요는 없으며, 동시에 여러 개의 그룹에 속하는 것도 가능하다.

```
// example
Group: Developers
  - Alice
  - Bob
  - Charles

Group: Operations
  - David
  - Edward

Group: Audit Team
  - Charles
  - David

User: (아무 그룹에도 속하지 않음)
  - Fred 
```

## 권한 (Permissions)
- **유저**와 **그룹**은 **정책(Policy)**이라고 불리는 JSON 문서로 할당될 수 있다.
- **정책(Policy)**은 곧 유저들이 갖춘 권한을 의미한다.
- AWS에서는 Least Privilege Principle (최소 권한의 원칙)을 적용하는데, 이는 곧 "이용자가 필요한 것 이상으로 권한을 부여하지 않음"을 의미한다.

### 정책 (Policy) 구조

```JSON
// EC2 정책에 대한 예시
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AttachVolume",
                "ec2:DetachVolume"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:volume/*",
                "arn:aws:ec2:*:*:instance/*"
            ],
            "Condition": {
                "ArnEquals": {"ec2:SourceInstanceARN": "arn:aws:ec2:*:*:instance/instance-id"}
            }
        }
    ]
}
```

- Version : 정책 언어의 버전, 거의 대부분 `2012-10-17`이 된다.
- Id : 정책에 대한 구분 명칭 (선택)
- Statement : 하나 이상의 독립적인 Statement (**필수**)
  - Sid : Statement에 대한 구분 명칭 (선택)
  - Effect : 특정 api 접근에 대한 허용/거부에 대한 여부 (Allow/Deny)
  - Principal : 해당 정책이 적용될 계정(account)/유저(user)/역할(role)
  - Action : 해당 정책이 허용하거나 거절할 액션 목록
  - Resource : 액션이 적용될 리소스의 목록
  - Condition : 해당 정책이 적용되는 조건 (선택)

## Password Policy (패스워드 정책)

- 강력한 패스워드 = 계정에 대해 더 강력한 보안
- AWS에서는 패스워드 정책을 갖추도록 할 수 있다.
  - 패스워드 최소 길이
  - 특정 타입의 문자 요구 ~ 대/소문자, 숫자, 특수문자 등...
  - IAM 유저가 패스워드를 본인 임의로 수정할 수 있도록 허용
  - 유저가 특정 기간 이후 패스워드를 변경하도록 요구 (password expiration)
  - 동일한 패스워드를 재사용할 수 없도록 함

## MFA ~ Multi Factor Authentication
- MFA = 알고 있는 패스워드 + 보유한 특정 보안 기기
- 패스워드와 MFA 토큰을 조합하여 로그인하도록 함
- **패스워드를 도용당하더라도, 계정은 보호할 수 있음**
- 이용 가능한 MFA 디바이스 옵션
  - Virtual MFA devices ~ Google Authenticator, Authy, 등...
    - 하나의 디바이스에 여러 토큰을 사용
  - Universal 2nd Factor (U2F) Security Key ~ YubiKey
    - 하나의 보안 키로 여러 루트 계정과 IAM 이용자들을 지원
  - Hardware Key Fob MFA Device
  - Hardware Key Fob MFA Device for AWS GovCloud

## AWS에 접근하는 여러가지 방법
- AWS Management Console ~ 패스워드 및 MFA로 보호
- AWS CLI (Command Line Interface) - Access Key로 보호
- AWS SDK (Software Developer Kit) - Access Key로 보호

### Access Key
- 액세스 키는 AWS Console을 통해 생성
- 유저는 각자 본인의 Access key를 관리하게 됨
- **Access Key는 패스워드와 마찬가지로, 비밀리에 관리되어야 하며, 공유해선 안된다.**
- Access Key ID ~= username
- Secret Access Key ~= password

## AWS CLI ?
- AWS 서비스를 커맨드라인 셸을 통해 상호작용할 수 있게끔 하는 툴
- AWS 서비스의 퍼블릭 api에 직접 접근할 수 있으며, 리소스들을 관리하는 별도의 스크립트를 작성할 수도 있다.
- 오픈 소스이며, AWS 관리 콘솔 대신 사용할 수도 있다.

## AWS SDK ?
- 특정 언어로 사용할 수 있는 API
- AWS 서비스를 프로그래밍적인 방식으로 접근 및 관리할 수 있음
- 애플리케이션 자체에 포함될 수 있다.
- 지원
  - SDKs, Mobile SDKs, IoT Device SDKs...
- Ex.) AWS CLI 자체가 AWS SDK for Python으로 작성된 것이다.

## AWS CloudShell
- AWS CLI가 탑재된 브라우저 기반의 셸
- 아직은 지역에 따라 지원 여부가 다름 (서울은 안 됨)

## IAM Roles for Services
- 일부 AWS 서비스는 특정한 동작을 수행할 필요가 있다.
- 이를 위해서, 이러한 AWS 서비스들이 이에 대한 권한을 가져야 하는데, 이것을 **IAM Roles**를 통해서 부여할 수 있다.
- 일반적인 Roles:
  - EC2 Instance Roles
  - Lambda Function Roles
  - Roles for CloudFormation

## IAM Security Tools
- IAM Credentials Report (account-level)
  - 루트 계정에 속한 모든 유저와 그들의 증명에 대한 다양한 상태에 대한 리포트
- IAM Access Advisor (user-level)
  - 한 유저에게 부여된 서비스 권한 및 그 유저가 접근한 서비스에 대해 보여줌
  - 정책을 개정하는 데 있어 해당 정보를 활용할 수 있음 (권한을 늘리거나 줄임)

## IAM Guidelines & Best Practices
- AWS 계정 설정을 위한 경우를 제외하고는 루트 계정을 사용하지 말 것
- 한 명의 실제 이용자 = 하나의 AWS User
- 여러 유저를 Group에 할당하고, 해당 그룹에 Permission을 부여함
- 강력한 패스워드 정책을 사용
- MFA를 사용
- AWS 서비스에 권한을 부여하기 위해 Role을 사용
- 프로그래밍적인 방식의 접근을 위해서는 Access Key를 사용 (CLI / SDK)
- IAM Credentials Report를 사용해 계정 권한을 점검
- **IAM 유저 및 Access Key는 절대 공유되어선 안됨**

## Summary
- Users: AWS 콘솔에 접근할 수 있는 이용자, password 부여
- Groups: 여러 User들이 포함
- Policies: 유저 및 그룹에 대한 권한에 대해 설명하는 JSON 문서
- Roles: EC2와 같은 AWS 서비스에 부여하는 권한
- Security: MFA + Password Policy
- Access Keys: AWS CLI or SDK에 사용되는 접근 키
- Audit: IAM Credential Reports & IAM Access Advisor