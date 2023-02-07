# ELB

## Scalability & High Availability

- Scalability는 상황에 따라 애플리케이션 / 시스템의 규모가 더 크게 조정할 수 있음을 의미함.
- 두 종류의 Scalability
  - Vertical Scalability
  - Horizontal Scalability (= elasticity)

- **Scalability는 High Availability와 관련이 있긴 하나, 엄연히 다른 개념**이다.

### Vertical Scalability

- 인스턴스 사이즈의 크기를 늘리는 것을 의미한다. (Ex. t2.micro -> t2.large)
- DB와 같은 비분산 시스템에 적용하는 것이 일반적이다. (Ex. RDS, ElastiCach)
- 확장에 있어 한계점이 있다. (하드웨어 제한)

### Horizontal Scalability

- 애플리케이션에 대한 인스턴스 또는 시스템의 갯수를 늘리는 것을 의미한다.
- 분산 시스템에 적용할 수 있다.
- 웹 애플리케이션 / 모던 애플리케이션에 매우 일반적으로 사용된다.

### High Availability

- High Availability는 주로 horizontal scaling과 일반적으로 관련이 있는 개념이다.
- 최소 2개의 데이터 센터(== AZ)에서 애플리케이션 / 시스템이 가동되고 있음을 의미한다.
- High Availability의 목표는 데이터 센터의 소실에서 살아남기 위함이다.
- High Availability는 수동적(passive)일 수도 있고(RDS Multi AZ), 능동적(active)일 수도 있다(Horizontal Scaling).

### High Availability & Scalability For EC2

- Vertical Scaling: 인스턴스 사이즈 증가 (= scale up / down)
- Horizontal Scaling: 인스턴스의 개수 증가 (= scale out / in)
  - Auto Scaling Group
  - Load Balancer
- High Availability: 여러 AZ를 통해 동일한 애플리케이션에 대한 인스턴스를 실행
  - Auto Scaling Group multi AZ
  - Load Balancer multi AZ

## Load Balancing

- Load Balances는 트래픽을 여러 downstream 서버(Ex. EC2 인스턴스)로 분산시켜주는 서버이다.

### Why use a load balancer?

- 부하(load)를 여러 다운스트림 인스턴스(downstream instance)로 분산시킬 수 있다.
- 애플리케이션에 대한 하나의 접근 지점(a single of access)를 노출시킬 수 있다.
- 다운스트림 인스턴스의 실패에 대해 유연하게(seamlessly) 대처할 수 있다.
- 인스턴스에 대한 정기 검진(regular health check)를 수행할 수 있다.
- 웹사이트에 SSL termination (HTTPS)을 제공할 수 있다.
- 쿠키를 통해 Session stickiness를 강화한다.
- 여러 AZ 너머로 High Availability를 갖는다.
- 퍼블릭 트래픽(Public traffic)을 프라이빗 트래픽(Private traffic)과 분리한다.

### Why use an Elastic Load Balancer?

- ELB(Elastic Load Balancer)는 **managed load balancer(관리된 로드 밸런서)**다.
  - AWS가 동작을 보장함
  - AWS가 업그레이드, 유지보수, 고가용성(High availability)를 관리한다.
  - AWS는 오직 소수의 configuration knob를 제공한다.
- 자체적인 로드 밸런서를 구축하는 경우, 비용은 더 적게 들지라도, 훨씬 더 많은 노력이 필요하다.
- AWS에서 제공하는 다양한 서비스들과 함께 통합되어 있다.
  - EC2, EC2 Auto Scaling Groups, Amazon ECS
  - AWS Certificate Manager (ACM), CloudWatch
  - Route 53, AWS WAF, AWS Global Accelerator

### Health Checks

- 헬스 체크는 로드 밸런서에 중요함.
- 로드 밸런서가 "트래픽을 전달받을 인스턴스가 요청에 응답할 수 있는지"에 대한 여부를 알 수 있도록 함.
- 헬스 체크는 하나의 포트와 하나의 루트에서 이루어짐. (일반적으로 `/health`)

### Types of load balancer on AWS

- AWS에는 **4가지 종류의 managed load balancer**가 있다.
  - Classic Load Balancer (v1 - old generation, 사라질 예정) - 2009 - CLB
    - HTTP, HTTPS, TCP, SSL (secure TCP)
  - Application Load Balancer (v2 - new generation) - 2016 - ALB
    - HTTP, HTTPS, WebSocket
  - Network Load Balancer (v2 - new generation) - 2017 - NLB
    - TCP, TLS (secure TCP), UDP
  - Gateway Load Balancer - 2020 - GWLB
    - Operates at layer 3 (Network layer) - IP Protocol

- 대체로는 새로운 세대의 로드 밸런서를 이용하는 것이 추천된다. 더 많은 기능을 보유하기 때문이다.
- 일부 로드 밸런서는 internal(private) 또는 external (public) ELB로써 설정될 수 있다.

## Application Load Balancer (v2)

- Application load balancer는 레이어 7(Application Layer)에 해당
- 여러 머신들로 여러 HTTP 애플리케이션에 대한 로드 밸런싱 (target groups)
- 동일한 머신 내 여러 애플리케이션에 대한 로드 밸런싱 (Ex: containers)
- HTTP/2와 웹 소켓을 지원
- 리다이렉트 지원 (from HTTP to HTTPS for example)
- 다른 타겟 그룹(target group)에 대한 라우팅 테이블
  - URL 내 path에 기반한 라우팅 (example.com`/users` & example.com`/posts`)
  - URL 내 hostname에 기반한 라우팅 (`one.example.com` & `other.example.com`)
  - 쿼리스트링, 헤더에 기반한 라우팅 (example.com/users?`id=123&order=false`)
- ALB는 마이크로 서비스 & 컨테이너 기반 애플리케이션에 매우 적합함. (ex. Docker & Amazon ECS)
- ECS의 다이나믹 포트로 리다이렉트 해주는 포트 매핑(port mapping) 기능이 있음.
- CLB(Classic Load Balancer)로 치면, 각각의 애플리케이션에 여러 개의 CLB를 두는 것과 유사함.

### Application Load Balancer (v2) ~ Target Groups

- 어떤 것들이 Target group이 될 수 있는가?
  - EC2 인스턴스 (Auto Scaling Group으로 관리될 수 있음) - HTTP
  - ECS tasks (ECS 자체적으로 관리됨) - HTTP
  - Lambda functions - HTTP 요청이 JSON 이벤트로 변환
  - IP Addresses - 반드시 private IP들이어야 함
- ALB는 여러 개의 target group에 라우팅을 할 수 있음
- 헬스 체크(health check)는 target group level에서 수행됨

### Application Load Balancer (v2) ~ Good to Know

- 호스트네임(hostname)이 고정됨 ~ (XXX.region.elb.amazonaws.com)
- 애플리케이션 서버는 클라이언트의 IP를 직접 볼 수 없음
  - 클라이언트의 진짜 IP는 `X-Forwarded-For` 헤더에 삽입됨
  - 포트(`X-Forwarded-Port`)와 프로토콜(`X-Forwarded-Proto`)도 알 수 있음

## Network Load Balancer (v2)

- Network load balancers (Layer 4)는
  - **인스턴스들에 TCP & UDP 트래픽을 포워딩**
  - 매초 수백만(million)의 요청을 처리할 수 있음
  - ALB(400ms)보다 더 낮은 레이턴시 (~100ms)
- ***각 AZ마다 하나의 정적 IP*를 가지며, Elastic IP 할당을 지원** (특정 IP에 대한 화이트리스팅(whitelisting)에 유용함)
- NLB는 극도의 성능과 함께 TCP 또는 UDP 트래픽을 다루어야 하는 경우에 사용됨.
- AWS 프리티어에 해당하지 않음.

### Network Load Balancer (v2) - Target Groups

- EC2 instances
- IP Addresses - 반드시 private IPs
- Application Load Balancer (ALB)
- **TCP, HTTP, HTTPS 프로토콜에 대한 헬스 체크를 지원**

## Gateway Load Balancer

- 3rd party network virtual appliance들을 배포, 확장, 관리할 수 있음
  - Ex.) Firewalls, Intrusion Detection and Prevention systems, Deep Packet Inspection Systems, payload manipulation
- Layer 3(Network Layer)에서 동작 - IP 패킷
- 아래 기능들을 합친 것
  - **Transparent Network Gateway** - 모든 트래픽에 대한 단일 entry/exit
  - **Load Balancer** - virtual appliance에 대한 트래픽 분산
- **6081 포트에 GENEVE 프로토콜을 사용**

### Gateway Load Balancer - Target Groups

- EC2 instances
- IP Addresses - must be private IPs

## Sticky Sessions (Session Affinity)

- stickiness를 구현하여 동일한 클라이언트는 로드 밸런서를 거치더라도 항상 동일한 인스턴스로 리다이렉트되도록 할 수 있음
- Classic Load Balancer & Application Load Balancer에서 사용 가능
- stickiness를 위해 사용되는 쿠키는 임의로 조정할 수 있는 expiration date가 존재
- 사례 : 이용자가 세션 데이터를 잃어버리지 않도록 해야하는 경우
- stickiness의 적용은 EC2 인스턴스의 부하에 불균형을 일으킬 가능성도 있음.

### Sticky Sessions - Cookie Names

- Application-based Cookies
  - Custom cookie
    - 타겟에 의해 생성
    - 애플리케이션에서 요구되는 커스텀 속성들을 추가할 수 있음
    - 각 타겟 그룹에 대해 독립적으로 정의되어야 함
    - **AWSALB**, **AWSALBAPP**, **AWSALBTG**는 사용할 수 없음 (ELB 자체적으로 사용됨)
  - Application cookie
    - 로드 밸런서에 의해 생성
    - **AWSALBAPP**이 쿠키명이 됨
- Duration-based Cookies
  - 로드 밸런서에 의해 생성되는 쿠키
  - ALB의 경우 **AWSALB**, CLB의 경우 **AWSELB**라는 쿠키명이 됨.

## Cross-Zone Load Balancing

![With CZLB](https://docs.aws.amazon.com/images/elasticloadbalancing/latest/userguide/images/cross_zone_load_balancing_enabled.png)

- Cross Zone Load Balancing을 이용하는 경우:
  - 각각의 로드 밸런서 인스턴스가 모든 AZ에 있는 모든 등록 인스턴스 너머로 균일하게 로드를 분산시킴

![Without CZLB](https://docs.aws.amazon.com/images/elasticloadbalancing/latest/userguide/images/cross_zone_load_balancing_disabled.png)

- Cross Zone Load Balancing을 이용하지 않는 경우:
  - Elastic Load Balancer의 노드 내에 있는 인스턴스들 내에서만 균일하게 분산시킴

- Application Load Balancer
  - 기본적으로 활성화되어 있음 (타겟 그룹 단위로 비활성화 가능)
  - AZ 간 데이터(inter AZ data)에 부과되는 비용 없음
- Network Load Balancer & Gateway Load Balancer
  - 기본적으로 비활성화
  - 활성화 시에는 AZ 간 데이터(inter AZ data)에 비용이 부과됨
- Classic Load Balancer
  - 기본적으로 비활성화
  - 활성화 시에는 AZ 간 데이터(inter AZ data)에 비용이 부과됨

## SSL/TLS - Basics

- SSL 인증서는 클라이언트와 로드밸런서 간의 트래픽을 전송 중에(in transit) 암호화해주는 역할을 한다. (in-flight encryption)
- **SSL** : Secure Socket Layer, 암호화 연결을 위해 사용
- **TLS** : Transport Layer Security, SSL의 새 버전
- 요즘은 **TLS 인증서가 주로 사용**되지만, 사람들은 여전히 이걸 SSL이라는 명칭으로 부른다.
- 공공 SSL 인증서는 Certificate Authorities(CA)에서 발급함
  - ex.) Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt, etc...
- SSL 인증서는 (직접 정하는) 만료일이 존재하며, 반드시 정기적으로 갱신되어야 한다.

### Load Balancer - SSL Certificates

- 로드 밸런서는 X.509 인증서를 사용함 (SSL/TLS server certificate)
- ACM(AWS Certificate Manager)를 통해서 인증서를 관리할 수 있음
- 원한다면 직접 본인이 소유한 인증서를 업로드할 수도 있음
- HTTPS listener:
  - 기본 인증서를 지정해야 함
  - 다중 도메인(multiple domains)을 지원하기 위해 선택적 인증서 목록(optional list of certs)을 추가할 수 있음
  - 클라이언트는 본인이 도달한 호스트 네임을 지정하기 위해 SNI (Server Name Indication) 을 사용할 수 있음
  - 구 버전의 SSL/TLS을 지원하고자 하는 경우, HTTPS에 특정 보안 정책을 설정할 수 있음 (legacy clients)

### SSL - Server Name Indication (SNI)

- SNI는 **하나의 웹 서버에서 여러 SSL 인증서들이 로드되는 문제를 해결**함 (여러 웹사이트를 서빙하기 위해)
- 이는 새로운 프로토콜이며, 클라이언트에게 처음 SSL 핸드셰이크를 할 때에 **타겟 서버의 호스트네임을 알려주는(indicate) 역할**을 함
- 클라이언트는 이를 바탕으로 올바른 인증서를 찾거나, 또는 기본 인증서를 반환함
- 유의점:
  - ALB와 NLB, 그리고 CloudFront에서만 동작 (신세대)
  - CLB에는 적용할 수 없음 (구세대)

## Elastic Load Balancers - SSL Certificates

- Classic Load Balancer (v1)
  - 오직 하나의 SSL 인증서만 지원
  - 여러개의 SSL 인증서를 가진 여러 개의 호스트네임을 위해서는 반드시 여러 개의 CLB를 사용해야만 함

- Application Load Balancer (v2)
  - 여러 개의 SSL 인증서를 가진 여러 개의 리스너를 지원함
  - 이것이 가능하게 하도록 SNI (Server Name Indication)을 사용함

- Network Load Balancer (v2)
  - 여러 개의 SSL 인증서를 가진 여러 개의 리스너를 지원함
  - 이것이 가능하게 하도록 SNI (Server Name Indication)을 사용함
