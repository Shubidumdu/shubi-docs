# EC2

- EC2 = Elastic Compute Cloud = Infrastructure as a Service (IaaS)
- EC2는 단순한 하나의 서비스일 뿐 아니라, 더 많은 서비스로 조합된다.
  - Virtual machine 대여 (EC2)
  - Virtual drive 내 데이터 저장 (EBS)
  - Virtual machin 간의 로드 분산 (ELB)
  - Auto-scaling group을 사용하는 서비스의 스케일링 (ASG)
- EC2의 이해는 곧 클라우드의 동작 방식을 이해하는 것의 기초가 된다.

## Sizing & Configuration

- 운영 체제 (OS): Linux, Windows or MacOS
- CPU
- RAM
- 저장 공간
  - 네트워크 결합 스토리지 : EBS & EFS
  - 하드웨어 : EC2 Instance Store
- Network card: 카드 속도 및 퍼블릭 IP 주소
- Firewall Rules : Security group
- Bootstrap script (최초 실행 시의 설정): EC2 User Data

### EC2 User Data

- EC2 User data 스크립트를 통해 인스턴스를 부트스트랩(bootstrap)할 수 있다.
- 스크립트는 인스턴스가 최초 실행될 때 단 한번만 실행된다.
- 다음과 같은 부트 시의 작업을 자동화할 수 있다.
  - 업데이트 설치
  - 소프트웨어 설치
  - 인터넷으로부터 파일 설치
  - 생각하는 뭐든지!
- EC2 User Data는 root user로 실행된다. (즉, 모든 권한을 갖는다.)

## EC2 Instance Types

- EC2에는 여러 유스케이스에 적용될 수 있는 다양한 타입의 EC2 인스턴스가 존재한다.
- 네이밍 컨벤션이 존재한다.

> m5.2xlarge
>
> - m : 인스턴스 클래스
> - 5 : 세대 (AWS 측에서 시간이 지남에 따라 이를 향상시킴)
> - 2xlarge : 인스턴스 클래스 내 사이즈

### Instance Types - General Purpose (Txx, Mxx)

- 웹 서버 또는 코드 저장소와 같이 다양한 작업 상황에서 활용할 수 있음
- 다음의 각각을 적절히 고려한 밸런스형 인스턴스 타입
  - Compute
  - Memory
  - Network

### Instance Types - Compute Optimized (Cxx)

- 높은 성능의 프로세서를 요구하는 연산 위주의 작업에 유용함.
  - Batch processing workloads
  - Media transcoding
  - High performance web servers
  - High performance computing (HPC)
  - Dedicated gaming servers

### Instance Types - Memory Optimized (Rxx, Xxx)

- 메모리 내에서 거대한 데이터 셋을 처리해야 하는 경우에 유용함.
  - High performance, relational/non-relational databases
  - Distributed web scale cache stores
  - In-memory databases optimized for BI (Business Intelligence)
  - Applications performing real-time processing of big unstructured data

### Instance Types - Storage Optimized (Ixx, Gxx, Hxx)

- 로컬 스토리지 내 거대한 데이터 셋에 접근해서 순차적인 읽기/쓰기를 처리해야 하는 상황에 유용함.
  - High frequency online transaction processing (OLTP) systems
  - Relational & NoSQL databases
  - Cache for in-memory databases (for example, Redis)
  - Data warehousing applications
  - Distributed file systems

## Security Groups

- Security Groups는 AWS의 네트워크 보안의 기초가 된다.
- EC2 인스턴스의 인/아웃바운드 트래픽을 어떻게 처리할지를 다룸.
- 오직 **allow** rule만 포함한다.
- Security Groups의 rule은 IP 주소 혹은 Security Group에 의해 참조될 수 있다.

### Security Groups Deep Dive

- Security Group은 곧 EC2 인스턴스의 **방화벽**처럼 동작한다.
- 다음을 다룰 수 있다.
  - 특정 포트에 대한 엑세스
  - 허용하는 IP의 범위 ~ IPv4 / IPv6
  - 인바운드 네트워크에 대한 통제 (from other to the instance)
  - 아웃바운드 네트워크에 대한 통제 (from the instance to other)
- 그 밖에 알면 좋은 것들
  - 여러 인스턴스들에 대해 적용될 수 있다.
  - 하나의 리전(region) / VPC에 격리된다.
  - EC2 **밖에** 존재하는 것이다. ~ 즉, Security Group 쪽에서 트래픽이 막히는 경우, EC2 측에서는 이에 대해 알 수 없다.
  - _SSH 엑세스만을 위한 별도의 Security group을 구축하는 편이 좋다._
  - 만약 앱에서 타임 아웃이 발생한다면, 아마도 Security group 이슈일 가능성이 크다.
  - 만약 앱에서 "connection refused" 에러가 발생한다면, 애플리케이션 자체의 에러이거나, 그것이 실행되지 않았을 가능성이 크다.
  - 인바운드 트래픽은 기본적으로 **막혀있다.(blocked)**
  - 아웃바운드 트래픽은 기본적으로 **권한을 갖는다.(authorized)**

### Classic Ports to know

- 22 = SSH (Secure Shell) - 리눅스 인스턴스로 로그인
- 21 = FTP (File Transfer Protocol) - 파일 공유를 위한 파일 업로드
- 22 = SFTP (Secure File Transfer Protocol) - SSH를 통한 파일 업로드
- 80 = HTTP - 안전하지 않은 웹사이트 접근
- 443 - HTTPS - 안전한 웹사이트 접근
- 3389 = RDP (Remote Desktop Protocol) - 윈도우즈 인스턴스로 로그인

## EC2 Instances Purchasing Options

- On-Demand Instances - short workload, predictable pricing, pay by second
- Reserved (1 & 3 years)
  - Reserved Instances - long workloads
  - Convertible Reserved Instances - long workloads with flexible instances
- Savings Plans (1 & 3years) - commitment to an amount of usage, long workload
- Spot Instances - short workloads, cheap, can lose instances (less reliable)
- Dedicated Hosts - book an entire physical server, control instance placement
- Dedicated Instances - no other customers will share your hardware
- Capacity Reservations - reserve capacity in a specific AZ(availability zone) for any duration

### On-Demand

- 쓴 만큼 지불
  - Linux or Windows : 첫 1분 이후 초당 비용
  - 그 외의 운영체제 : 시간 당 비용
- 가장 비용이 높으나, 선불(upfront) 금액이 없음
- 장기 계약(long-term commitment)이 없음
- 즉, **짧은 기간(short-term)** 동안, **중단되어선 안되는 작업(un-interrupted workloads)**을 수행해야 하는 경우에 추천됨 ~ 앱이 어떻게 동작할지 예측할 수 없는 경우.

### Reserved Instances

- On-demand 대비 72% 만큼 비용 절감
- 특정 인스턴스 속성에 대해 예약 (Instance type, Region, Tenancy, OS)
- 예약 기간 - 1년 또는 3년 (길수록 더 크게 절감)
- 결제 옵션 - No Upfront -> Partial Upfront -> All Upfront 순으로 비용 절감
- 예약 인스턴스 범위(Scope) - Regional or Zonal (하나의 AZ 내에 종속)
- 꾸준히 사용되어야 하는 앱에 추천됨 (Ex. DB)
- Reserved Instance Marketplace에서 사거나 팔 수 있음
- **Convertible Reserved Instance**
  - 인스턴스 type, family, OS, scope, tenancy 등을 변경할 수 있음

### Savings Plans

- 장 기간의 사용에 대한 할인 (최대 72% ~ Reserved Instances와 동일)
- 특정량 만큼의 사용을 약속 (Ex. 1년 또는 3년 동안 시간 당 $10를 쓸 것임)
- On-Demand로 청구된 가격을 지불할 때 적용됨.
- 특정 Instance family와 AWS region에 제한됨. (Ex. M5 in us-east-1)
  - 단, 다음에 대해선 유연하게 변경 가능
    - 인스턴스 사이즈 (Ex. m5.xlarge, m5.2xlarge)
    - OS (Ex. Linux, Windows)
    - Tenancy (Host, Dedicated, Default)

### Spot Instances

- On-demand 대비 최대 90%까지 절감
- 단, 설정한 최대 비용보다 현재 spot price가 더 비싸지는 경우에 언제든지 인스턴스가 사라질 수 있음.
- AWS에서 가장 비용 절감이 많이 되는 형태의 인스턴스
- **실패하는 경우에도 문제없는 탄력적인(resilient) 작업에 유용함.**
  - Batch jobs
  - Data analysis
  - Image processing
  - Any distributed workloads
  - Workloads with a flexible start and end time
- **중요한 작업이나 DB의 경우엔 적합하지 않음.**

### Dedicated Hosts

- 특정 이용자만 사용할 수 있는 전용 EC2 인스턴스에 대한 물리적 서버를 제공
- **Compliance requirements**를 준수하고, **기존에 보유한 Server-bound software licenses**(소켓 당, 코어 당, VM 당)를 사용할 수 있도록 허용.
- 구매 옵션
  - On-demand : 활성화된 Dedicated Host에 대해 초 당 비용
  - Reserved : 1년 또는 3년 (No Upfront, Partial Upfront, All Upfront)
- 가장 비싼 인스턴스 옵션임.
- 복잡한 라이센싱 모델(Complicated licensing model)을 갖춘 소프트웨어의 경우에 유용함
- 또는, 강한 규제 및 법률을 보유한 기업의 경우에 사용함.

### Dedicated Instances

- 이용자에게 종속된 하드웨어에서 동작하는 인스턴스
- 동일한 계정 내 다른 인스턴스들과도 하드웨어를 공유할 수 있음
- 인스턴스의 위치에 대한 통제권은 없음 (인스턴스의 실행/종료마다 하드웨어가 변경될 수 있음)
- Dedicated host와 비교했을 때, dedicated host는 말 그대로 물리적 서버 자체에 대한 접근권을 갖기 때문에 보다 저수준(low-level)의 하드웨어에 대한 가시성(visibility)을 갖는 반면, Dedicated Instance의 경우는 그렇지 못한다는 점에서 차이가 생김.

### Capacity Reservations

- 특정 AZ의 특정 기간 동안에 On-demand 인스턴스 용량(capacity)를 예약
- 필요한 경우 언제든 EC2 가용량에 접근할 수 있음
- **별도의 계약이 없으며(언제든 생성/취소할 수 있음), 별도의 비용 절감도 없음**
- Reserved Instance와 Saving Plans와 결합하여 비용 절감을 취할 수 있음
- 인스턴스가 실제로 실행되든 아니든 간에 비용이 On-demand로 청구됨.
- 짧은 기간 동안, 중단되어선 안되는 작업(uninterrupted workload)이 특정 AZ 내에서 수행되어야 하는 경우에 적합.

## Spot Instance

- On-demand 대비 90%까지 비용 절감 가능한 인스턴스 옵션
- **max spot price**를 지정 후, 해당 인스턴스의 **현재 spot price가 지정한 max spot price보다 작은 경우**에 해당 인스턴스를 보유할 수 있게 됨.
  - 매 시간마다 spot price는 수요/공급(offer/capacity)을 고려해서 달라짐.
  - 만약, 보유할 수 없는 상황이 된 경우, 2분 동안의 유예 기간(grace period)를 부여받으며, 해당 인스턴스를 중지(stop)할 것인지, 종료(terminate)할 것인지 선택할 수 있음.
- Spot Block ~ 특정 시간대에 대해 적용하는 블록(block) spot instance. 지금은 없어짐.
- 실패해도 무방한 작업들, 즉 내결함성(resilient)을 갖춘 작업들의 경우에 적합함. 반대로, 중요한 작업이나 DB에는 부적합.

### Spot instance request를 종료하고자 하는 경우

![Spot Instance Lifecycle](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/spot_lifecycle.png)

![Spot Instance State](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/spot_request_states.png)

- open, active, disabled 상태에서만 Spot instance request을 취소할 수 있음
- Spot instance request의 취소가 곧 인스턴스의 종료(terminate)를 의미하진 않음
- 따라서, Spot instance를 종료하고 싶은 경우, 반드시 먼저 spot request를 취소한 다음, 그 다음에 관련된 Spot instance를 종료해야 함.
  
### Spot Fleets

- Spot Fleets = set of Spot Instances + (선택) On-Demand Instances
- 주어진 비용 제한(price constraints)에 맞춰 목표 용량(target capacity)을 충족하려고 시도함.
  - 가능한 launch pools 정의 : 인스턴스 타입, OS, AZ
  - 여러 개의 launch pools를 가질 수 있으며, 이렇게 하면 fleet이 선택할 수 있게 됨
  - Spot Fleet은 용량 또는 최대 비용에 도달하는 경우 인스턴스 실행을 멈춤
- Spot instances 할당 전략
  - lowerPrice: 가장 비용이 저렴한 pool부터 시도 (cost optimization, short workload)
  - diversified: 모든 pool에 대해 광범위하게 적용 (great for availability, long workloads)
  - capacityOptimized: 인스턴스 개수 대비 최적의 용량을 가진 pool부터 시도
- **Spot Fleet은 Spot Instance에 대한 요청을 최소화된 비용으로 자동으로 처리할 수 있도록 해줌.**

## Elastic IPs

- EC2 인스턴스를 중지/실행할 때마다, 퍼블릭 IP가 변경될 수 있다.
- 만약, 인스턴스의 퍼블릭 IP를 고정시키고자 한다면, Elastic IP가 필요하다.
- Elastic IP는 퍼블릭 IPv4 IP에 해당하며, 삭제하지 않는 이상 유지된다.
- 한번에 하나의 인스턴스에 대해서만 부여할 수 있다.
- Elastic IP 주소를 사용하면, 인스턴스 또는 소프트웨어에 문제가 발생했을 때, 해당 주소를 다른 인스턴스에 부여해서 문제에 대처할 수 있다.
- 단, Elastic IP 주소는 한 계정 당 오직 5개까지만 가질 수 있다. (AWS에 더 달라고 요청할 수는 있다.)
- 결론적으로, **Elastic IP를 쓰는 것을 피하는 것이 좋다.**

  - 일반적으로는 좋지 않은 결정이다.
  - 대신에, 랜덤 퍼블릭 IP를 사용하고, 그것에 대한 DNS 네임을 등록하는 편이 좋다.
  - 또는, 로드 밸런서를 사용하면서 퍼블릭 IP 자체를 사용하지 않는 방법도 있는데, 이 방법이 가장 최적이다.

### Private vs Public IP in AWS EC2

- 기본적으로, EC2 머신에서
  - 프라이빗 IP는 AWS 내부 네트워크를 위해서,
  - 퍼블릭 IP는 WWW를 위해서 사용된다.

- 만약, EC2 머신에 SSH 연결을 시도한다면
  - (VPN이 구축되어 있지 않다면) 프라이빗 IP가 아닌, 퍼블릭 IP를 써야한다. 동일한 네트워크에 놓여있는 것이 아니기 때문.

- 머신이 중지/재실행된다면, **퍼블릭 IP는 바뀔 수 있다.**

## Placement Groups

- EC2 인스턴스의 배치 전략(placement strategy)을 조정하고 싶을 때, **Placement group**을 정의할 수 있다.
- Placement group은 다음의 전략 중 하나를 선택할 수 있다.
  - Cluster - 하나의 AZ 내에 low-latency group으로 인스턴스들을 밀집
  - Spread - 소규모 인스턴스 그룹을 다른 기본 하드웨어(underlying hardware)로 분산하여 오류를 줄임 (각 AZ의 그룹 당 최대 7개 인스턴스), 중요한 애플리케이션에서 활용.
  - Partition - 인스턴스들을 하나의 AZ 내의 파티션 별로 다른 기본 하드웨어로 분산하여 오류를 줄임.

### Placement Groups - Cluster

![Cluster](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/placement-group-cluster.png)

- 장점 : 높은 네트워크 성능 (인스턴스 간 latency가 작음)
- 단점 : 하드웨어 상 결함이 발생한다면, 모든 인스턴스가 다 같이 실패함.
- 사례
  - 빠르게 수행되어야 하는 거대한 데이터 기반의 작업
  - 극도로 낮은 latency와 높은 네트워크 throughput을 보장해야하는 애플리케이션

### Placement Groups - Spread

![Spread](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/placement-group-spread.png)

- 장점
  - 동일한 지역의 여러 AZ로 확장할 수 있음
  - 동시에 인스턴스들이 실패할 위험을 줄일 수 있음
- 단점
  - 각 Placement group 내 하나의 AZ에 최대 7개의 인스턴스로 제한됨.
- 사례
  - 최대한 높은 가용성(availability)을 확보해야 하는 애플리케이션
  - 인스턴스 서로가 각자의 동작 실패로부터 격리되어야 하는 중요한 애플리케이션

### Placement Groups - Partition

![Partition](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/placement-group-partition.png)

- AZ 당 최대 7개의 파티션
- 동일한 지역 내 여러 AZ로 확장 가능
- 최대 100개의 인스턴스를 파티션 설정에 사용할 수 있음
- 하나의 파티션 내에 있는 인스턴스들은 다른 파티션의 인스턴스들과 rack을 공유하지 않음
- 하나의 파티션 내에서 발생한 문제는 해당 파티션에 속한 인스턴스들에는 영향을 끼칠 수 있으나, 그 외의 파티션에는 영향을 끼치지 않음.
- EC2 인스턴스들은 메타데이터를 통해 파티션 정보에 접근할 수 있음
- **사례** : HDFS, HBase, Cassandra, Kafka

## Elastic Network Interfaces (ENI)

- **가상의 네트워크 카드(virtual network card)를 나타내는 VPC 내 논리적 네트워킹 컴포넌트**
- ENI는 다음과 같은 속성을 갖출 수 있음
  - 기본(primary) 프라이빗 IPv4, 하나 또는 그 이상의 보조(secondary) IPv4
  - 하나의 프라이빗 IPv4 당 하나의 Elastic IP (IPv4)
  - 하나의 퍼블릭 IPv4
  - 하나 이상의 Security groups
  - Mac address
- ENI를 독립적으로 생성하고, EC2 인스턴스가 실패했을 때에 대비하여 인스턴스에 적용(attach)한다.
- 하나의 특정한 AZ에 격리된다.

## EC2 Hibernate

- 인스턴스의 중지(stop) / 종료(terminate)
  - **Stop** - 디스크에 저장된 데이터(EBS)가 다음 인스턴스의 실행까지 유지됨
  - **Termninate** - 삭제 될 것으로 설정된 특정 EBS 볼륨(root)은 사라짐
- 인스턴스의 실행 시, 다음과 같은 일이 일어난다.
  - 최초 실행 시 : OS 부팅 & EC2 User Data Script가 실행됨
  - 그 이후의 실행 시 : OS 부팅
  - 위의 각 단계 이후에 애플리케이션이 실행되고, 캐시가 웜업(warm-up) 된다.

### EC2 Hibernate 란?

- in-memory(RAM) 상태를 보존할 수 있음
- 인스턴스 부팅이 훨씬 빨라짐 (OS가 중지되거나 재실행되지 않음)
- 원리 : RAM의 상태가 루트 EBS 볼륨에 하나의 파일로 작성되어 보존.
  - 즉, 루트 EBS 볼륨이 반드시 암호화되어야 함.
- 사례 :
  - 오래 동안 진행되어야 하는 프로세싱
  - RAM 상태를 저장해야 하는 경우
  - 초기화에 시간이 많이 걸리는 서비스
- 그 밖의 특징
  - 지원하는 인스턴스 패밀리 - C3, C4, C5, I3, M3, M4, R3, R4, T2, T3, ...
  - 인스턴스 RAM 사이즈 - 150GB 미만 이어야 함
  - 인스턴스 사이즈 - 베어 메탈 인스턴스(bare metal instance)에는 지원하지 않음
  - AMI - Amazon Linux 2, Linux AMI, Ubuntu, RHEL, CentOS & Windows, ...
  - 루트 볼륨 - 반드시 다음의 속성을 갖춘 EBS여야 함
    - encrypted
    - not instance store
    - large
  - On-Demand, Reserved, Spot 인스턴스 모두에 적용 가능.
  - 인스턴스는 60일을 초과해서 hibernate 할 수 없음.

## EC2 Instance Store

- EBS 볼륨은 **네트워크 드라이브**에 해당하여 좋은 점도 있지만, **제한된 성능**을 갖는다.
- **만약, 고성능의 하드웨어 디스크가 필요한 상황이라면, EC2 Instance Store의 사용을 고려해야 한다.**
- 더 나은 I/O 성능을 제공
- EC2 Instance Store는 인스턴스가 멈추게 되면 데이터가 사라진다. (ephemeral = 일시적임)
- buffer / cache / scratch data / temporary content에 유용함
- 하드웨어 문제가 발생하는 경우 데이터가 사라질 수 있음
- 백업 및 복제(replication)는 이용자가 직접 처리해야 함
