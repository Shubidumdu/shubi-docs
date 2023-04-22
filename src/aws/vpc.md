# Networking - VPC

## Understanding CIDR - IPv4

- **CIDR(Classless Inter-Domain Routing)** - IP 주소 할당 방법
- **Security Groups** Rule과 AWS 네트워킹에서 일반적으로 사용됨
- IP 주소의 범위를 정의하는 것을 도와줌
  - `WW.XX.YY.ZZ/32` -> 단일 IP
  - `0.0.0.0/0` -> 모든 IP
  - `192.168.0.0/26` -> `192.168.0.0` ~ `192.168.0.63` (64개의 IP 주소)

- CIDR은 두 부분으로 나뉨
  - **Base IP**
    - 범위에 포함될 IP (`XX.XX.XX.XX`)
    - ex. `10.0.0.0`, `192.168.0.0`, ...
  - **Subnet Mask**
    - IP 내에서 얼마나 많은 bit가 바뀔 수 있는지를 정의
    - ex. `/0`, `/24`, `/32`
    - 두 가지 형태가 될 수 있음
      - `/8` <-> `255.0.0.0`
      - `/16` <-> `255.255.0.0
      - `/24` <-> `255.255.255.0`
      - `/32` <-> `255.255.255.255`

- Subnet Mask는 기본적으로 기본 IP로부터 얼마나 추가적인 값들을 허용할지 나타내는 부분이다.
  - 예시
    - `192.168.0.0/32` => 1개의 IP 허용 (2^0) => `192.168.0.0`
    - `192.168.0.0/31` => 2개의 IP 허용 (2^1) => `192.168.0.0` ~ `192.168.0.1`
    - `192.168.0.0/24` => 256개의 IP 허용 (2^8) => `192.168.0.0` ~ `192.168.0.255`
    - `0.0.0.0/0` => 모든 IP 허용 (2^32) => `0.0.0.0` ~ `255.255.255.255`

- 주로 사용되는 경우

  - `/32` - 어떤 octet도 바뀔 수 없음
  - `/24` - 마지막 octet이 바뀔 수 있음
  - `/16` - 뒤의 두 octet이 바뀔 수 있음
  - `/8` - 뒤의 세 octet이 바뀔 수 있음
  - `/0` - 모든 octet이 바뀔 수 있음

![Octet Example](https://f4n3x6c5.stackpathcdn.com/article/getting-started-with-vpc-virtual-private-cloud-part2/Images/1.png)

- 헷갈린다면 CIDR를 IP 주소 범위로 변환해주는 사이트도 있으니 [참고](https://www.ipaddressguide.com/cidr)

## Public vs. Private IP (IPv4)

- IANA(Internet Assigned Numbers Authority)가 public / private 주소 사용을 위해 특정 IPv4 주소 블록을 설정해뒀음
- **Private IP**는 다음의 특정 값들만 허용됨:
  - `10.0.0.0` ~ `10.255.255.255` (`10.0.0.0/8`) -> 대규모 네트워크
  - `172.16.0.0` ~ `172.31.255.255` (`172.16.0.0/12`) -> **AWS 기본 VPC의 범위**
  - `192.168.0.0` ~ `192.168.255.255` (`192.168.0.0/16`) -> ex. 홈 네트워크
- 인터넷 상에서 그 외의 나머지 IP 주소들은 모두 Public IP

## Default VPC Walkthrough

- 모든 새 AWS 계정들은 기본 VPC를 가짐
- 새로운 EC2 인스턴스들은 별도로 서브넷을 지정하지 않는다면 기본 VPC로 실행됨
- 기본 VPC는 인터넷 연결이 되어있고, 그 안의 모든 EC2 인스턴스들은 public IPv4 주소를 갖게 됨
- public/private IPv4 DNS 네임을 가질 수 있음

## VPC in AWS - IPv4

- **VPC - Virtual Private Cloud**
- 하나의 AWS 리전 내에 여러 VPC들을 둘 수 있음 (리전 별 최대 5개 ~ soft limit)
- VPC 별 CIDR은 최대 5개이며, 각 CIDR의 경우:
  - 최소 사이즈 `/28` (16개의 IP 주소)
  - 최대 사이즈 `/16` (65536개의 IP 주소)
- VPC는 private이기 때문에, 오직 Private IPv4 주소만 허용됨:
  - `10.0.0.0` ~ `10.255.255.255` (`10.0.0.0/8`)
  - `172.16.0.0` ~ `172.31.255.255` (`172.16.0.0/12`)
  - `192.168.0.0` ~ `192.168.255.255` (`192.168.0.0/16`)
- **내 VPC CIDR은 내 다른 네트워크와 겹쳐선 안됨 (ex. 회사의 IP 주소)**

## VPC - Subnet (IPv4)

- AWS 리소스들은 각 서브넷 별로 **5개의 IP 주소 (first 4 & last 1)**를 예약해둠
- 이 5개의 IP 주소는 사용할 수 없고, EC2 인스턴스에 할당될 수도 없음
- 예시: 만약 CIDR 블록이 `10.0.0.0/24`라면, 예약되는 IP 주소들은 다음과 같음
  - `10.0.0.0` - Network Address
  - `10.0.0.1` - AWS에 의해 예약, VPC 라우터 용도
  - `10.0.0.2` - AWS에 의해 예약, Amazon에서 제공하는 DNS에 매핑
  - `10.0.0.3` - AWS에 의해 예약, 추후 사용 용도
  - `10.0.0.255` - Network Broadcast Address로, AWS에서는 VPC에서 broadcast를 지원하지 않기 때문에, 예약되어 있음
- **시험 팁**: 만약 EC2 인스턴스에 29개의 IP 주소가 필요하다면?
  - `/27` 사이즈의 서브넷을 둘 수 없음 (32개 IP 주소 ~ 32 - 5 = 27 < 29)
  - `/26` 사이즈의 서브넷을 골라야 함 (64개 IP 주소 ~ 64 - 5 = 59 > 29)

## Internet Gateway (IGW)

- VPC 내에 있는 리소스(e.g., EC2 인스턴스)들의 인터넷 연결을 허용
- 수평적 확장 & highly available & 중복(redundant)
- 반드시 VPC와는 별도로 생성되어야 함
- 하나의 VPC는 오직 하나의 IGW에 연결될 수 있으며, 반대의 경우도 마찬가지
- Internet Gateway 그 자체만으로는 인터넷 액세스를 허용하지 않음
  - 반드시 Route table도 함께 수정되어야 함!

![Internet Gateway Example](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/internet-gateway-basics.png)

## Bastion Hosts

- private EC2 인스턴스에 SSH 연결을 사용하고 싶을 때 Bastion Host를 사용할 수 있음
- bastion은 다른 private 서브넷들에 연결되어 있는 public 서브넷
- **Bastion Host security group은 반드시 제한된 CIDR의 22번 포트로부터 인바운드만 허용해야 함**
  - ex. 회사의 **public CIDR**
- **EC2 인스턴스의 Security Group**은 반드시 Bastion Host의 Security Group을 허용하거나, Bastion host의 **private IP**를 허용해야 함

![Bastion Host](https://cloudacademy.com/wp-content/uploads/2015/11/aws-bastion-host-1.png)

## NAT Instance (**outdated**, but still at the exam)

- **NAT = Network Address Translation**
- private 서브넷 내의 EC2 인스턴스들이 인터넷에 연결될 수 있도록 해줌
- 반드시 public 서브넷 내에서 실행되어야 함
- 다음 EC2 설정을 반드시 비활성화 해야함: **Source / destination Check**
- 연결할 Elastic IP가 반드시 필요함
- private 서브넷으로부터의 트래픽을 NAT 인스턴스로 라우팅하도록 Route Table이 반드시 설정되어야 함

![NAT Instance Example](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/nat-instance_updated.png)

### NAT Instance - Comments

- 사전에 설정되어 있는(pre-configured) Amazon Linux AMI를 사용할 수 있음
  - 공식 지원은 2020/12/31에 끝났음
- 기본적으로는 Highly Available 및 resilient setup이 제공되지 않음
  - multi-AZ에 ASG를 만들고, resilient user-data script를 생성할 필요가 있음
- 인터넷 트래픽 대역폭(bandwidth)는 EC2 인스턴스 유형에 따라 다름
- 반드시 Security group & Rule을 관리해야함:
  - 인바운드:
    - private 서브넷으로부터 오는 HTTP/HTTPS 트래픽을 허용
    - 홈 네트워크로부터의 SSH 연결(Internet Gateway를 통한 액세스)을 허용
  - 아웃바운드:
    - 인터넷으로의 HTTP/HTTPS 트래픽을 허용

## NAT Gateway

- AWS가 관리하는 NAT, 높은 대역폭, high availability, 관리 불필요
- 사용한 시간과 대역폭에 따라 비용 지불
- NAT Gateway는 특정 AZ에 생성되며, Elastic IP를 사용
- 동일한 서브넷 내 EC2 인스턴스에서 사용될 수는 없음 (다른 서브넷을 통해서만 가능)
- IGW(Internet Gateway)가 필요함 (Private Subnet => NATGW => IGW)
- 5Gbps의 대역폭, 최대 45Gbps로 스케일 업
- 따로 Security Group을 관리해야 하거나 필요하지 않음

### NAT Gateway with High Availability

- **NAT Gateway는 단일 AZ 내에서 resilient함**
- 내결함성(fault-tolerance)를 위해서는 반드시 **여러 AZ** 내에 **여러 NAT Gateway**들을 생성해야함
- cross-AZ failover가 필요 없음 -> AZ가 다운되면 NAT가 필요하지 않기 때문

### NAT Gateway vs. NAT Instance

- [참고](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)

## Network Access Control List (NACL)

- NACL은 **서브넷** 안팎으로 오가는 트래픽을 관리하는 일종의 방화벽
- **서브넷 별로 하나의 NACL**이 있고, 새로운 서브넷에는 **Default NACL**이 할당됨
- **NACL Rules**를 정의:
  - Rule에는 번호가 지정(1~32766), 낮은 번호일 수록 더 높은 우선순위
  - 첫번째로 일치하는 규칙을 기반으로 결정을 내림
  - ex. 만약 #100번 규칙으로 `10.0.0.10/32`를 **허용**했다면, #200번 규칙으로 `10.0.0.10/32`에 대해 **거부**했다면, 해당 IP 주소에 대해서는 허용될 것임
  - 마지막 Rule은 아스터리스크(`*`)이며, 아무런 규칙도 매칭되지 않을 경우 요청을 거부
  - AWS에서는 100 단위로 규칙을 추가할 것을 추천함
- 새로 생성된 NACL의 경우 모든 경우에 대해 거부
- NACL은 서브넷 레벨에서 특정 IP 주소를 블록하기에 아주 좋은 방법

### Default NACL

- 할당된 서브넷에 대한 모든 인바운드/아웃바운드를 허용함
- **Default NACL을 수정하지 말 것**, 대신에 커스텀 NACL을 새로 생성해야함

### Ephemeral Ports (임시 포트)

- 두 엔드포인트 간에 연결이 이루어지려면, 반드시 포트를 사용해야 함
- 클라이언트가 **defined port**(= fixed port)로 연결하고, **ephemeral port**(임시 포트)에서 응답을 내보냄
- 운영체제가 다르다면, 포트의 범위도 다르게 사용함
  - 예시
    - IANA & MS Windows 10 -> 49152 ~ 65535
    - Many Linux Kernels -. 32768 ~ 60999

### NACL with Ephemeral Ports

- [참고](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports)

### Securiy Group vs. NACLs

![NACL vs SG](https://clouds.geant.org/wp-content/uploads/2019/01/security6.png)

## VPC Peering

- AWS 네트워크를 통해 두 VPC 간에 private하게 연결을 할 수 있게 해줌
- 마치 두 VPC가 동일한 네트워크에 있는 것처럼 동작하게 만듬
- 중복되는 CIDR을 가져선 안됨
- VPC Peering 연결은 **transitive**(전이적)하지 않음 (반드시 VPC 간에 서로 직접 연결되어야 함)
- **EC2 인스턴스들이 서로 상호작용할 수 있도록 하려면 각각의 VPC 서브넷 안에 있는 Route Table을 업데이트 해야만 함**

![VPC Peering](https://docs.aws.amazon.com/images/whitepapers/latest/aws-vpc-connectivity-options/images/image14.png)

### VPC Peering - Goot to know

- **서로 다른 AWS 계정/리전** 내에서도 VPC 간에 VPC Peering 연결을 생성할 수 있음
- 피어링된 VPC 내에서는 Security Group을 서로 참조할 수 있음 (동일한 리전 내 cross account 가능)

## VPC Endpoints (AWS PrivateLink)

- 모든 AWS 서비스는 공개적으로 노출 (public URL)
- VPC Endpoints (powered by AWS PrivateLink)는 public 인터넷 대신 **private network**를 사용하여 AWS 서비스들을 연결할 수 있도록 해줌
- redundant, 수평적 확장
- AWS 서비스들에 액세스 하기 위해 IGW, NATGW 같은 것들을 사용할 필요가 없어짐
- 문제가 발생했을 경우:
  - VPC에서 DNS 설정을 확인
  - Route Table을 확인

![VPC Endpoints](https://docs.aws.amazon.com/images/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/images/aws-privatelink.png)

## Types of Endpoints

- **Interface Endpoints (powered by PrivateLink)**
  - ENI(private IP 주소)를 엔트리 포인트(반드시 Security Group 연결 필요)로 프로비저닝
  - 대부분의 AWS 서비스들을 지원
  - 시간 당 비용 + 처리되는 데이터 GB당 비용
- **Gateway Endpoints**
  - 게이트웨이를 프로비저닝하며, **반드시 이를 Route Table 내에서 타겟으로 사용해야 함** (**Security Group 사용이 필요 없음**)
  - S3와 DynamoDB를 지원
  - 무료

### Gateway or Interface Endpoint for S3?

- **Gateway가 거의 대부분의 경우 더 선호됨**
- 비용 측면: Gateway Interface는 무료인 반면 Interface Endpoint는 비용 필요
- Interface Endpoint는 온-프레미스 혹은 다른 VPC/리전으로부터의 액세스가 필요한 경우에 선호됨

## VPC Flow Logs

- 내 인터페이스로 이동하는 IP 트래픽들에 대한 정보를 캡처:
  - VPC Flow Logs
  - Subnet Flow Logs
  - Elastic Network Interface (ENI) Flow Logs
- 모니터링 & 연결 이슈에 대한 트러블슈팅에 도움
- Flow Logs 데이터는 S3, CloudWatch Logs, Kinesis Data Firehose로 보낼 수 있음
- AWS에서 관리되는 인터페이스로부터의 네트워크 정보도 캡처:
  - ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway..

### VPC Flow Logs Syntax

![VPC Flow Logs Syntax](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2019/08/13/2019-08-13_10-41-04.png)

- **srcaddr & dstaddr** - 문제가 있는 IP를 파악하는 데에 도움
- **srcport & dstport** - 문제가 있는 포트를 파악하는 데에 도움
- **Action** - Security Group / NACL에 따른 요청 성공/실패 여부
- 사용 패턴 / 악의적 행동에 대한 분석을 위해 사용할 수 있음
- S3의 Athena 또는 CloudWatch Log Insights를 통해 VPC Flow Log를 쿼리할 수 있음
- [Flow Log 예시](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html)

### VPC Flow Logs - Troubleshoot SG & NACL Issues

- **ACTION 필드를 확인하자!**
  - **Incoming Request**의 경우
    - 인바운드 REJECT => NACL 또는 SG
    - 인바운드 ACCEPT, 아웃바운드 REJECT => NACL
  - **Outgoing Requests**의 경우
    - 아웃바운드 REJECT => NACL 또는 SG
    - 아웃바운드 ACCEPT, 인바운드 REJECT => NACL

### VPC Flow Logs - Architectures

- VPC Flow Logs -> CloudWatch Logs -> CloudWatch Contributor Insights
- VPC Flow Logs -> CloudWatch Logs -> CloudWatch Alarm -> Amazon SNS
- VPC Flow Logs -> S3 Bucket -> Amazon Athena -> Amazon QuickSight

## AWS Site-to-Site VPN

- **Virtual Private Gateway (VGW)**
  - VPN 연결 시 AWS 측의 VPN concentrator
  - VGW를 생성 후 Site-to-Site VPN 연결을 생성하고 싶은 VPC에 연결
  - ASN(Autonomous System Number)를 커스터마이징 할 수 있음
- **Customer Gateway (CGW)**
  - VPN 연결의 customer 측 소프트웨어 애플리케이션 또는 [실제 디바이스](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)
  
### Site-to-Site VPN Connections

- **Customer Gateway Devices (On-Premises)**
  - **어떤 IP 주소를 사용하는가?**
    - Customer Gateway 디바이스에 대한 Public Internet-routable IP 주소를 사용
    - 만약 NAT traversal(NAT-T)가 활성화된 NAT 디바이스라면, NAT 디바이스의 public IP 주소를 사용
- **중요한 과정**: 내 서브넷과 연결된 Route Table 내 Virtual Private Gateway에 대한 **Route Propagation**을 활성화해야 함
- 온-프레미스로부터 EC2 인스턴스를 ping해야 한다면, Security Group의 인바운드로 ICMP 프로토콜을 추가하였는지 확인해야 함

![Site-to-Site VPN Connection](https://docs.aws.amazon.com/ko_kr/vpn/latest/s2svpn/images/vpn-basic-diagram.png)

### AWS VPN CloudHub

- 여러 VPN 연결을 보유한 경우, 여러 사이트 간의 보안 연결을 제공
- 다른 로케이션 간의 주요 또는 보조 네트워크 연결을 위한 저비용 hub-and-spoke 모델 (VPN Only)
- VPN 연결이므로, public 인터넷을 통해 연결됨
- 설정하려면 동일한 VGW에 여러 VPN을 연결하고, dynamic routing을 설정하고 Route Table을 구성

![AWS VPN CloudHub](https://docs.aws.amazon.com/ko_kr/vpn/latest/s2svpn/images/AWS_VPN_CloudHub-diagram.png)

## Direct Connect (DX)

- 원격 네트워크에서 내 VPC로 전용 **private** 연결을 할 수 있게 해줌
- 전용 연결은 반드시 내 Direct Connect와 AWS Direct Connection 로케이션 사이에 구성되어야 함
- VPC에 Virtual Private Gateway를 설정해야 함
- 하나의 연결로 public 리소스(S3)와 private 리소스(EC2)에 액세스할 수 있음
- 사례:
  - 대역폭 처리량 증가 - 거대한 데이터 셋을 다루는 경우 ~ 더 낮은 비용
  - 보다 일관적인 네트워크 경험 - 실시간 데이터 피드를 사용하는 애플리케이션
  - 하이브리드 환경 (온-프레미스 + 클라우드)
- IPv4와 IPv6 모두 지원

### Direct Connect Diagram

![Direct Connect](https://docs.aws.amazon.com/ko_kr/directconnect/latest/UserGuide/images/direct-connect-overview.png)

### Direct Connect Gateway

- **여러 리전(동일한 계정) 간에 하나 이상의 VPC로 Direct Connect를 설정하고자 한다면 반드시 Direct Connect Gateway를 사용해야 함**

![Direct Connect Gateway](https://docs.aws.amazon.com/ko_kr/directconnect/latest/UserGuide/images/dx-gateway.png)

### Direct Connect - Connection Types

- **Dedicated Connections**: 1Gbps, 10Gbps and 100Gbps 가용량
  - customer 전용의 물리적 이더넷 포트
  - AWS에 먼저 요청을 하고, 이후 AWS Direct Connect Partner로부터 완료됨

- **Hosted Connections**: 50Mbps, 500Mbps, 최대 10Gbps
  - AWS Direct Connect Partner를 통해 연결 요청이 이루어짐
  - 가용량(capacity)이 **on-demand로 추가/삭제**될 수 있음
  - AWS Direct Connect Partner 선택에 따라 1, 2, 5, 10Gbps

- 새로운 연결을 구축하는데는 주로 1달 이상 소요됨

### Direct Connect - Encryption

- 데이터는 전송 중(in transit) **암호화 되지 않지만**, private함
- AWS Direct Connect + VPN은 IPsec-encrypted된 private 연결을 제공함
- 더 추가적인 보안을 챙길 수 있으나, 구축에 있어 좀 더 복잡함

![AWS Direct Connect + VPN](https://docs.aws.amazon.com/ja_jp/whitepapers/latest/aws-vpc-connectivity-options/images/image10.png)

### Direct Connect - Resiliency

- **주요 워크로드에 대한 High Resiliency**
  - 여러 location들에 대해 하나로 연결

![Direct Connect One connection at multi location](https://d1.awsstatic.com/AWS%20Direct%20Connect/Redundancy-Higher.cf97732b85470f0b3616f025f8a3534f95bb1940.png)

- **주요 워크로드에 대한 Maximum Resiliency**
  - 하나 이상의 location 내 별도의 장치에서 종료되는 연결들을 분리함으로써 Maximum resilience를 지킬 수 있음

![Direct Connect Maximum Resiliency for Critical Workloads](https://d1.awsstatic.com/AWS%20Direct%20Connect/Redundancy-Highest.cc18117d65de87b62bf4b8e10db7f980e3ac13fd.png)

### Site-to-Site VPN connection as a backup

- Direct Connect가 실패할 경우를 대비하고자 하는 경우
  - 백업 Direct Connect 연결을 구축하거나 (비쌈)
  - Site-to-Site VPN 연결을 구축할 수 있음

## Transit Gateway

- **수천 개의 VPC와 온-프레미스 사이에 transitive(전이적) peering을 구축함으로써 hub-and-spoke(star) 연결을 만듬**
- 리전 별 리소스, cross-region으로 동작할 수 있음
- RAM(Resource Access Manager)를 통해 계정 간에 공유 가능
- 리전 간에 Transit Gateway들을 피어링할 수 있음
- Route Tables: 다른 VPC와 소통할 수 있는 VPC들을 제한할 수 있음
- Direct Connect 게이트웨이 및 VPC 연결과 호환 가능
- **IP Multicast** 지원 (다른 어떤 AWS 서비스에서도 지원하지 않음)

![Transit Gateway](https://docs.aws.amazon.com/images/whitepapers/latest/building-scalable-secure-multi-vpc-network-infrastructure/images/hub-and-spoke-design.png)

### Transit Gateway - Site-to-Site VPN ECMP

- **ECMP = Equal-Cost Multiple-Path routing
- 여러 최적의 경로를 통해 패킷을 넘기도록 하는 라우팅 전략
- 사례: **AWS로의 연결 대역폭을 상승시키기 위해** 여러 개의 Site-to-Site VPN 연결을 구축

![Site-to-Site VPN ECMP](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2020/02/02/Single-Tunnel.png)

### Transit Gateway - throughput with ECMP

- VPN to Virtual Private Gateway
  - 하나의 터널로 하나의 VPC에 연결
  - 하나의 터널 -> 1.25Gbps
- VPN to Transit Gateway
  - 하나의 site-to-site VPN 연결로 여러 개의 VPC에 연결
  - 하나의 site-to-site VPN 연결은 2.5Gbps (ECMP ~ 해당 전략에 두개의 터널이 사용됨)
    - 더 많은 site-to-site VPN 연결을 추가할수록 처리량이 증가
      - 2개 ~ 5.0Gbps (ECMP)
      - 3개 ~ 7.5Gbps (ECMP)
  - Transit Gateway를 통해 처리되는 데이터 GB 당 비용 지불

### Transit Gateway - Share Direct Connect between multiple accounts

![Share Direct Connect between multiple accounts](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2021/05/26/updated-Diagram321-Transit-VIF1.png)

## VPC - Traffic Mirroring

- 내 VPC 내에서의 네트워크 트래픽을 캡처 및 검사하도록 해줌
- 내가 관리하는 보안 appliance로 트래픽을 라우팅
- 트래픽 캡처
  - **From (Source)** - 여러 ENI
  - **To (Targets)** - 하나의 ENI 또는 하나의 NLB
- 모든 패킷을 캡처 또는 내가 관심있는 패킷만 캡처 (선택적으로 패킷 잘라내기 가능 ~ truncate)
- Source와 Target은 동일한 VPC에 있을수도, 다른 VPC에 있을 수도 있음 (VPC Peering)
- 사례: 컨텐츠 검사, 위협 모니터링, 트러블슈팅...

![VPC Traffic Mirroring](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2021/03/08/Slide1.jpeg)

## IPv6 for VPC

- IPv4는 4.3 billon(43억)개의 주소를 제공 (곧 소진될 것)
- IPv6는 IPv4의 후속
- IPv6는 **3.4 x 10^38** 개의 고유 IP 주소를 제공
- 모든 IPv6 주소는 public이며, internet-routable함 (= private 범위가 없음)
- 형태 => `x.x.x.x.x.x.x.x` (*x*는 16진수, 범위는 0000 ~ ffff)
- 예시:
  - `2001:db8:3333:4444:5555:6666:7777:8888`
  - `2001:db8:3333:4444:cccc:dddd:eeee:ffff`
  - `::` => 모든 8 segments가 모두 0
  - `2001:db8::` => 뒤의 6 segments가 모두 0
  - `::1234:5678` => 첫 6 segments가 모두 0
  - `2001:db8::1234:5678` => 가운데 4 segments가 모두 0

### IPv6 in VPC

- **IPv4는 VPC와 서브넷에서 비활성화될 수는 없음**
- dual-stack 모드로 작동시키기 위해 IPv6를 활성화할 수는 있음 (모두 public IP 주소)
- 내 EC2 인스턴스들에서는 최소한 내부 private IPv4와 public IPv6를 갖게 됨
- Internet Gateway으로 인터넷과 IPv4 또는 IPv6 양측을 통해 소통할 수 있게됨

![IPv6 in VPC](https://docs.aws.amazon.com/images/whitepapers/latest/ipv6-on-aws/images/internet-access-for-a-vpc-public-subnet.png)

### IPv6 Troubleshooting

- **IPv4는 VPC와 서브넷에서 비활성화될 수는 없음**
- 따라서, 내 서브넷에서 EC2 인스턴스를 실행할 수 없다면
  - 이는 IPv6를 획득할 수 없기 때문이 아니라 (IPv6 공간은 매우 크기 때문)
  - 서브넷 내에서 이용가능한 IPv4이 없기 때문임
- **해결책**: 내 서브넷에서 새로운 IPv4 CIDR을 생성
