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
