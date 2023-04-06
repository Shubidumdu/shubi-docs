# Elastic Beanstalk

## Instantiating Applications quickly

- EC2 인스턴스
  - **Golden AMI 사용**: 애플리케이션 설치, OS 의존성 등을 미리 처리한 Golden AMI로부터 바로 EC2 인스턴스를 실행
  - **User Data를 사용한 부트스트랩**: 동적인 환경 설정에 대해서는 User Data 스크립트를 사용
  - **Hybrid**: Golden AMI와 User Data를 함께 사용 (Elastic Beanstalk)
- RDS 데이터베이스
  - 스냅샷으로부터 복구: 동일한 스키마와 데이터를 가진 데이터베이스를 만들 수 있음
- EBS 볼륨
  - 스냅샷으로부터 복구: 이미 디스크가 포맷되고, 데이터를 가진 상태로 시작

## Developer problems on AWS

- 인프라(infrastructure) 관리
- 코드 배포
- 데이터베이스, 로드 밸런서 등의 설정
- 스케일링
  
- 대부분의 웹앱은 동일한 아키텍처를 가짐 (ALB + ASG)
- 모든 개발자들이 원하는 것은 단순히 직접 작성한 코드가 실행되길 원함
- 또, 여러 다른 애플리케이션과 환경을 오고가는 상황에서, 내 애플리케이션을 한가지 방법으로 쉽게 배포하고자 할 수 있음

## Elastic Beanstalk - Overview

- Elastic Beanstalk는 AWS에서의 애플리케이션 배포에 있어 개발자를 위한 서비스
- 앞서 봤던 모든 컴포넌트들을 사용함: EC2, ASG, ELB, RDS, ...
- 관리형 서비스
  - 다음의 것들을 자동으로 처리해줌 - 용량 프로비저닝(capacity provisioning), 로드 밸런싱, 스케일링, 애플리케이션 헬스 모니터링, 인스턴스 설정, ...
  - 개발자가 신경쓸 것은 단순히 애플리케이션 코드 뿐임
- 여전히 설정에 대한 모든 통제는 갖고 있음
- Beanstalk 자체는 무료지만, 여기서 사용되는 인스턴스들에 대한 가격은 지불함

## Elastic Beanstalk - Components

- **Application**: Elastic Beanstalk 컴포넌트의 집합 (환경, 버전, 설정, ...)
- **Application Version**: 애플리케이션 코드의 iteration
- **Environment**
  - 애플리케이션 버전을 작동시키는 AWS 리소스들의 집합 (오직 한번에 하나의 애플리케이션 버전만)
  - **Tiers**: 웹 서버 환경 티어 & 워커 환경 티어
  - 여러 개의 환경을 만들 수 있음 (dev, test, prod, ...)

### Elastic Beanstalk - Worker environments

![Worker environment](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/aeb-architecture_worker.png)

## Elastic Beanstalk - Supported Platforms

- Go, Java, .NET, Node.js 등등등..
- Packer Builder
- Single Container Docker
- Multi Container Docker
- Preconfigured Docker
- 만약, 해당하는 게 없다면 별도로 커스텀 플랫폼을 작성할 수도 있음 (고급)

## Elastic Beanstalk - Deployment Modes

- Single Instance - dev 환경에 좋음
- High Availability with Load Balancer - prod 환경에 좋음
