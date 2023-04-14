# Containers

## Docker

### Docker - What is Docker?

- Docker는 앱을 배포하기 위한 소프트웨어 개발 플랫폼
- 앱이 **컨테이너**에 패키지되어 어떤 OS에서든지 실행 가능함
- 앱은 어디서 실행하든 동일하게 실행됨
  - 어떤 머신이든 동작
  - 호환성 이슈 없음
  - 예측 가능한 동작
  - 작업 감소
  - 유지 및 배포가 쉬움
  - 어떤 언어/OS/기술과도 호환됨
- 사례:
  - 마이크로서비스 아키텍처
  - 온-프레미스에서 AWS 클라우드로 앱을 리프트-앤-시프트(lift-and-shift)

### Docker - Where are Docker images stored?

- 도커 이미지들은 도커 저장소(repositories)에 저장됨
  - **Docker Hub** (<https://hub.docker.com>)
    - **Public** repo
    - 여러 기술 또는 OS를 위한 베이스 이미지들을 사용 (e.g., Ubuntu, MySQL, ...)
  - **Amazon ECR** (**Amazon Elastic Container Registry**)
    - **Private** repo
    - **Public** repo (**Amazon ECR Public Gallery - <https://gallery.ecr.aws>)

### Docker - Docker vs. Virtual Machines

- Docker는 "일종의" 가상화 기술이긴 하지만, 정확하게는 다름
- Resources가 호스트와 공유됨 => 하나의 서버에 여러 컨테이너가 존재

![Docker vs VM](https://i.ytimg.com/vi/TvnZTi_gaNc/maxresdefault.jpg)

### Docker - Docker Containers Management os AWS

- **Amazon Elastic Container Service (Amazon ECS)**
  - 아마존의 자체적인 컨테이너 플랫폼
- **Amazon Elastic Kubernetes Services (Amazon EKS)**
  - 아마존의 관리형 쿠버네티스 (오픈 소스)
- **AWS Fargate**
  - 아마존의 자체적인 서버리스 컨테이너 플랫폼
  - ECS, EKS와 호환
- **Amazon ECR**
  - 컨테이너 이미지를 저장

## Amazon ECS

![ECS](https://d1.awsstatic.com/product-page-diagram_Amazon-ECS%402x.0d872eb6fb782ddc733a27d2bb9db795fed71185.png)

### ECS - EC2 Launch Type

- ECS = Elastic Container Service
- AWS에 도커 컨테이너를 실행 = ECS 클러스터에서 **ECS Task**를 실행
- **EC2 Launch Type**: **반드시 인프라(EC2 인스턴스)를 프로비전 & 유지 해야함**
- 각 EC2 인스턴스는 반드시 ECS Agent를 실행하여 ECS 클러스터에 등록해야 함
- AWS가 컨테이너의 시작/중지를 맡아서 함

### ECS - Fargate Launch Type

- AWS에 도커 컨테이너를 실행
- **인프라를 프로비전하지 않음** (**EC2 인스턴스를 관리할 필요 없음**)
- **서버리스**!
- task를 정의하기만 하면 됨
- 내가 필요한 CPU / RAM에 따라 AWS는 단순히 ECS Task를 실행해줄 뿐임
- 스케일링을 위해서는 그냥 task의 갯수를 늘리면 됨, 쉬움 -> EC2 인스턴스 필요 없음

### ECS - IAM Roles for ECS

- **EC2 Instance Profile (EC2 Launch Type only)**:
  - **ECS agent**가 사용함
  - ECS 서비스를 위한 API 호출
  - CloudWatch 로그에 컨테이너 로그를 전송
  - ECR로부터 도커 이미지를 가져옴
  - Secret Manager 또는 SSM Parameter Store에 있는 민감한 데이터를 참조

- **ECS Task Role**:
  - 각 task에 특정한 role이 부여되도록 허용
  - 각기 다른 ECS 서비스에 다른 role들을 사용
  - Task Role은 **task definition**을 통해 정의됨

### ECS - Load Balancer Integrations

- **Application Load Balancer(ALB)**가 지원되며, 거의 대부분의 상황에 적합함
- **Network Load Balancer(NLB)**는 높은 처리량 / 높은 성능이 요구되는 사례에 추천, 또는 AWS Private Link와 함께 쓰기 위해 사용
- Elastic Load Balancer 역시 지원되지만, 추천되지는 않음 (고급 기능을 사용할 수 없음 - Fargate 불가)

### ECS - Data Volumes (EFS)

- ECS task에 EFS 파일 시스템을 마운트
- **EC2**와 **Fargate** launch type 모두와 호환
- 어떤 AZ에서 task가 실행되든 간에 EFS 파일 시스템 내에서 동일한 데이터를 공유함
- **Fargate + EFS = Serverless**
- 사례: 내 컨테이너에 대해 multi-AZ 간에 공유되는 스토리지를 지속해야 하는 경우
- 노트: S3는 파일 시스템으로 마운트될 수 없음

### ECS - Auto Scaling

- 자동으로 ECS task의 desired number를 증가/감소
- Amazon ECS Auto Scaling은 **AWS Application Auto Scaling**을 사용
  - ECS Service 평균 CPU 사용률
  - ECS Service 평균 메모리 사용률 - RAM에 따른 스케일링
  - 각 타겟 별 ALB 요청 수 - ALB로부터 통계 전달받음
- **Target Tracking** - 특정 CloudWatch 통계의 target value에 기반하여 스케일링
- **Step Scaling** - 특정 CloudWatch Alarm에 기반하여 스케일링
- **Scheduled Scaling** - 특정 날짜/시간에 기반하여 스케일링 (예측 가능한 변경의 경우)
- ECS Service Auto Scaling (task 레벨) != EC2 Auto Scaling (EC2 인스턴스 레벨)
- Fargate Auto Scaling은 훨씬 설정이 쉬움 (**서버리스**이기 때문)

### ECS - Launch Type ~ Auto Scaling EC2 Instances

- 기반에 놓인 EC2 인스턴스를 추가함으로써 ECS Service Scaling을 수용
- **Auto Scaling Group Scaling**
  - CPU 사용량에 기반하여 ASG를 스케일링
  - 시간이 지남에 따라 EC2 인스턴스 추가
- **ECS Cluster Capacity Provider**
  - ECS task를 위한 인프라에 대한 프로비저닝과 스케일링을 자동으로 처리하기 위해 사용
  - Auto Scaling Group과 짝지어서 사용
  - 용량이 부족할 때 EC2 인스턴스를 추가 (CPU, RAM...)

### ECS - Solutions Architectures

![ECS with EventBridge](https://practical-aws.dev/p/container-ecs-event-bridge-scheduled/group-container-ecs-event-bridge-scheduled-architecture.png)

![ECS with SQS](https://imgur.com/6YLbKsU.png)

## ECR

- ECR = Elastic Container Registry
- AWS에 도커 이미지를 저장 및 관리
- **Private** 또는 **Public** repo (**Amazon ECR Public Gallery** ~ <https://gallery.ecr.aws>)
- ECS와 완전히 호환됨, S3 지원
- IAM을 통해 액세스 관리 (권한 에러 => policy 문제)
- 이미지 취약성 스캐닝(image vulnerability scanning), 버저닝, 이미지 태그, 이미지 라이프사이클 지원

## Amazon EKS

### EKS - Overview

- EKS = Elastic **Kubernetes** Service
- **AWS에서의 관리형 쿠버네티스 클러스터를 실행하는 방법**
- 쿠버네티스?
  - 컨테이너형(보통은 Docker) 애플리케이션을 자동 배포, 스케일링, 관리해주기 위한 오픈소스 시스템
- ECS의 대체제로 쓰일 수 있고, 유사한 목표를 지녔으나 API가 다름
- EKS는
  - **EC2**로 워커 노드를 배포할 수도 있고
  - **Fargate**로 서버리스 컨테이너 배포를 할 수도 있음
- **사례**:
  - 온-프레미스 또는 다른 클라우드를 통해서 이미 쿠버네티스를 사용하고 있어 이를 AWS에 마이그레이션하고자 하는 경우
- **쿠버네티스는 클라우드에 구애받지 않음(cloud-agnostic)** (어떤 클라우드에서건 사용될 수 있음 - Azure, GCP, ...)

### EKS - Diagram

![EKS Diagram](https://d1.awsstatic.com/partner-network/QuickStart/datasheets/amazon-eks-on-aws-architecture-diagram.64cf0e40c45ade8107daf6a3ef5e2e05134d9a4b.png)

### EKS - Node Types

- **Managed Node Groups**
  - 노드(EC2 인스턴스)를 생성 및 관리해줌
  - 노드는 EKS가 관리하는 ASG의 일부
  - On-Demand 또는 Spot Instance 지원
- **Self-Managed Nodes**
  - 노드를 직접 생성하여, EKS 클러스터에 등록하고, ASG가 이를 관리하도록 함
  - prebuilt AMI를 사용할 수 있음 - Amazon EKS Optimized AMI
  - On-Demand 또는 Spot Instance 지원
- **AWS Fargate**
  - 별도의 유지관리 필요 없음; 노드 관리가 필요 없음

### EKS - Data Volumes

- EKS 클러스터에 **StorageClass** manifest를 정의할 필요 있음
- Container Storage Interface(CSI) 호환 드라이버를 활용함
- 다음을 지원
  - Amazon EBS
  - Amazon EFS (Fargate 지원)
  - Amazon FSx for Lustre
  - Amazon FSx for NetApp ONTAP

## AWS App Runner

- 웹 앱과 API 규모에 맞게 쉡개 배포해주는 완전 관리형 서비스
- 별도의 인프라 경험이 없어도 됨
- 소스 코드 또는 컨테이너 이미지만 있으면 됨
- 자동으로 웹 앱을 빌드 & 배포
- 다음 내용들을 자동으로 처리
  - 스케일링
  - 고가용성(HA)
  - 로드 밸런서
  - 암호화
- VPC 액세스 지원
- DB, 캐시, 메시지 큐 서비스와 연결됨
- 사례: 웹앱, API, 마이크로서비스, 빠른 프로덕션 배포
