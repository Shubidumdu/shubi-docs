# Route 53

## What is DNS?

- DNS은 IP 주소를 인간 친화적인 형태의 호스트네임으로 변환해주는 역할을 한다.
- ex.) `www.google.com` -> `172.217.18.36`
- DNS는 인터넷의 백본(backbone)이다.
- DNS는 계층적 네이밍 구조를 사용한다.

## DNS Terminologies

![DNS Terminologies](https://miro.medium.com/v2/resize:fit:1102/1*neC9vjKw-9WVVjHJy03ZIw.png)

- Domain Registrar: Amazon Route 53, GoDaddy, ...
- DNS Records: A, AAAA, CNAME, NS, ...
- Zone File: DNS 레코드를 포함
- Name Server: DNS 쿼리를 처리 (Authoritative or Non-Authoritative)
- Top Level Domain (TLD): .com, .us, .in, .gov, .org, ...
- Second Level Domain (SLD): amazon.com, google.com, ...

## How DNS Works

![How DNS Works](https://d1.awsstatic.com/Route53/how-route-53-routes-traffic.8d313c7da075c3c7303aaef32e89b5d0b7885e7c.png)

## Amazon Route 53

- highly available하고, scalable한, 완전 관리형 권한 보유(authoritative) DNS
  - authoritative = 고객(나)이 DNS 레코드를 갱신할 수 있음
- Route53은 또한 Domain Registrar이기도 함.
- 보유한 리소스에 대한 health check가 가능
- 100% availability SLA를 제공하는 유일한 AWS 서비스
- 왜 Route53이란 이름일까? -> 53은 전통적인 DNS 포트넘버

### Route 53 - Records

- 레코드 -> 도메인에 대한 트래픽을 라우팅하는 방법
- 각각의 record는 다음을 보유함
  - **Domain/subdomain Name** -> ex. example.com
  - **Record Type** -> ex. A or AAAA
  - **Value** -> ex. 12.34.56.78
  - **Route Policy** -> Route53이 쿼리에 대응하는 방식
  - **TTL** -> DNS resolver들에 레코드가 캐시되는 시간

- Route53은 아래의 DNS 레코드 타입들을 지원함
  - (**필수**) A / AAAA / CNAME / NS
  - (고급) CAA / DS / MX / NAPTR/ PTR / SOA / TXT / SPF/ SRV

### Route 53 - Record Types

- **A** - 호스트네임을 IPv4로 매핑
- **AAAA** - 호스트네임을 IPv6로 매핑
- **CNAME** - 호스트네임을 또 다른 호스트네임으로 매핑
  - 매핑한 대상은 반드시 A 또는 AAAA 레코드를 갖고 있어야 함
  - CNAME을 DNS 네임스페이스의 최상위 노드에서 생성할 수는 없음 (Zone Apex)
  - ex. `example.com`에서는 만들수 없지만, `www.example.com`에서는 만들 수 있음
- **NS** - Hosted Zone에 대한 네임 서버
  - 한 도메인에 대한 트래픽을 어떻게 라우트할지 통제함

### Route 53 - Hosted Zones

- 하나의 도메인과 그것의 서브도메인에 대한 트래픽을 어떻게 라우팅할 것인지 정의하는 레코드들의 컨테이너
- **Public Hosted Zones** - 인터넷(public domain names)에서 트래픽을 어떻게 라우팅할지 명시하는 레코드들을 보유 -> ex. `application1.mypublicdomain.com`
- **Private Hosted Zones** - 하나 또는 그 이상의 VPC(private domain names)에서 트래픽을 어떻게 라우팅할지 명시하는 레코드들을 보유 -> ex. `application1.company.internal`
- 각 hosted zone마다 한달에 $0.50 만큼 지불

### Route 53 - Records TTL (Time To Live)

- High TTL - ex. 24hr
  - Route 53에 낮은 트래픽을 전달
  - Record가 최신의 것이 아닐 가능성이 있음
- Low TTL - ex. 60sec
  - Route53에 더 많은 트래픽 (그에 따라 더 많은 비용)
  - Record를 최신으로 유지하기 쉬움 -> Record 변경이 쉬움
- **Alias 레코드를 제외하면, TTL은 각각의 DNS 레코드에 의무적으로 필요함**

### CNAME vs Alias

- AWS 리소스(Load Balancer, CloudFront...)들은 AWS 호스트 네임을 갖고 있음
  - `lbl-1234.us-east-2.elb.amazonaws.com`
- CNAME:
  - 하나의 호스트네임을 다른 호스트네임을 가리키도록 함 (`app.mydomain.com` -> `blabla.anything.com`)
  - **오직 루트가 아닌 도메인에 대해서만 사용 가능 (aka. somthing.mydomain.com)**
- Alias:
  - 하나의 호스트네임을 AWS 리소스를 가리키도록 함 (`app.mydomain.com` -> `blabla.amazonaws.com`)
  - **루트 도메인 여부와 상관없이 사용 가능 (aka. mydomain.com)**
  - 비용 청구 없음
  - 자체적인 헬스 체크

### Route 53 - Alias Records

- AWS 리소스에 대한 호스트네임을 매핑
- DNS 기능에 대한 확장
- 자동으로 AWS 리소스의 IP 주소 변경을 감지
- CNAME과 다르게, DNS 네임스페이스의 최상위 노드 (Zone Apex)에서도 사용할 수 있음 -> e.g: `example.com`
- Alias 레코드는 항상 AWS 리소스를 가리키는 A/AAAA 타입이어야 함 (IPv4 / IPv6)
- **TTL을 설정할 수 없음** ~ Route 53 자체적으로 알아서 처리

### Route 53 - Alias Records Targets

- Elastic Load Balancers
- CloudFront Distributions
- API Gateway
- Elastic Beanstalk environment
- S3 Websites
- VPC Interface Endpoints
- Global Accelerator accelerator
- 동일한 hosted zone 내의 Route53 record
- **EC2 DNS 네임을 ALIAS 레코드로 지정할 수는 없음**

## Route 53 - Routing Policies

- Route 53이 DNS 쿼리에 어떻게 응답할지 정의
- "Routing"이라는 명칭으로 인해 혼동하지 말 것
  - LB 관점에서의 트래픽 라우팅과는 다름
  - DNS는 어떤 트래픽도 라우트하지 않음, 단지 DNS 쿼리에 응답을 해줄 뿐임
- Route 53은 아래의 Routing 정책들을 지원함
  - Simple
  - Weighted
  - Failover
  - Latency based
  - Geolocation
  - Multi-Value Answer
  - Geoproximity (using Route 53 Traffic Flow feature)

### Routing Policies - Simple

- 일반적으로, 트래픽을 하나의 리소스로 라우팅
- 동일한 레코드에 여러 값을 정의할 수도 있음
- **만약, 여러 값이 정의된 경우, 클라이언트에 의해 무작위 값 하나가 선택됨**
- Alias가 활성화된 경우, 하나의 AWS만 정의할 수 있음
- 헬스 체크에 연결할 수 없음

### Routing Policies - Weighted

- 각각의 특정 리소스마다 요청 비율을 관리
- 각각의 레코드에 상대적 가중치를 할당 (합계가 꼭 100이어야 할 필요 없음)

$$ traffic(\%) = \frac{weight \space for \space a \space specific \space record}{Sum \space of \space all \space the \space weight \space for \space all \space records}$$

- DNS 레코드들은 반드시 동일한 이름과 타입을 지녀야 함
- 헬스 체크에 연결할 수 있음
- 유스 케이스: 리전 간의 로드 밸런싱, 새로운 애플리케이션 버전에 대한 테스트
- **가중치를 0으로 할당하는 경우, 레코드는 해당 리소스로는 트래픽을 전송하지 않음**
- **만약 모든 레코드의 가중치가 0이라면, 모든 레코드들이 동일한 가중치로 반환됨**

### Routing Policies - Latency-based

- 이용자에게 가장 적은 레이턴시를 제공할 수 있는(아마도 가장 가까운) 리소스로 리다이렉트
- 이용자들에 대한 레이턴시 보장이 최우선인 경우 매우 유용
- **레이턴시는 이용자와 AWS 리전 사이의 트래픽에 근거하여 결정됨**
  - e.g.) 독일인 이용자는 아마 US로 리다이렉트될 것 (만약 그쪽이 제일 레이턴시가 낮다면)
- 헬스 체크에 연결될 수 있음 (failover capability ~ 장애조치 기능)

### Routing Policies - Failover (Active-Passive)

- 먼저 주 인스턴스(primary)에 헬스체크를 시도한 이후, 그것이 실패하면 장애 조치 목적의 보조 인스턴스(secondary)로 라우팅

### Routing Policies - Geolocation

- Latency-based와는 차이가 있음
- **이용자의 위치 자체에 근거하여 결정됨**
- 대륙, 국가, 또는 주(미국)으로 위치를 정의 (겹치는 상황이라면, 가장 좁은 범위로 선택됨)
- **기본 레코드**를 생성하는 편이 좋음 (해당하는 위치가 없는 경우)
- 사례 - 웹사이트 localization, 컨텐츠 배포를 제한, 로드 밸런싱, ...
- 헬스 체크와 연결 가능

### Geoproximity Routing Policies - Geoproximity

- 이용자와 리소스의 지리적인 위치에 기반하여 트래픽을 라우팅
- 정의된 **바이어스**(bias)에 기반하여 **더 많은 트래픽을 각 리소스에 할당**
- 지리학적 지역의 크기를 변경하려면, 바이어스의 값을 정의
  - 확장(expand)하고자 하는 경우 (1 ~ 99): 해당 리소스에 더 많은 트래픽
  - 축소(shrink)하고자 하는 경우 (-1 ~ -99): 해당 리소스에 더 적은 트래픽

- 리소스는 아래와 같은 것이 될 수 있음
  - AWS 리소스 (AWS 리전 명시)
  - Non-AWS 리소스 (위도, 경도 명시)
- 해당 기능을 사용하기 위해서는 반드시 Route 53 Traffic Flow(고급)를 사용해야 함

### Routing Policies - IP-based Routing

- **클라이언트의 IP 주소에 기반하여 라우팅**
- **클라이언트의 CIDR와 엔드포인트/로케이션 목록을 제공** (이용자 IP-엔드포인트 매핑)
- 사례: 성능 최적화, 네트워크 비용 감축
- e.g., 특정 ISP로부터의 엔드 유저를 특정 엔드포인트로 라우팅

### Routing Policies - Multi-Value

- 트래픽을 여러 개의 리소스에 라우팅하고자 할 때 사용
- Route 53이 여러 개의 값/리소스를 반환
- 헬스 체크 연결 가능 (healthy resources에 대한 값들만 반환해줌)
- 각각의 Multi-Value 쿼리에 대해 최대 8개까지의 healthy record만 반환됨
- **Multi-Value가 ELB를 대체할 수는 없음**

## Route 53 - Health Checks

- HTTP 헬스 체크는 오직 **public 리소스** 들에만 적용 가능
- 헬스 체크 -> 자동화된 DNS 장애 조치:

  1. 엔드포인트에 대한 모니터링 (애플리케이션, 서버, 그 외 AWS 리소스)
  2. 다른 헬스 체크에 대한 모니터링 (Calculated Health Checks)
  3. CloudWatch 알람(완전 제어)에 대한 모니터링 - e.g., DynamoDB 쓰로틀링, RDS 알람, 커스텀 메트릭스, ... (private 리소스들의 경우에 유용)

- 헬스체크는 CloudWatch metrics와 함께 사용될 수 있음

### Health Checks - Monitor an Endpoint

- **약 15개의 글로벌 헬스 체커가 엔드포인트 헬스 상태를 체크**
  - Healthy/Unhealthy Threshold - 3 (default)
  - Interval - 30초 (10초로도 지정 가능 - 더 높은 비용)
  - 지원 프로토콜: HTTP, HTTPS, TCP
  - 약 18% 이상의 헬스 체커가 엔드포인트가 healthy 하다고 리포트한다면, Route 53은 이를 **Healthy**하다고 간주, 그렇지 않다면, **Unhealthy**
  - Route 53이 헬스 체크에 사용할 위치(location)이 어디인지 선택할 수 있음
- 헬스 체크는 오직 엔드포인트가 2xx, 3xx 상태 코드를 응답할 때만 통과
- 헬스 체크는 응답의 첫 **5120 bytes**의 텍스트에 기반하여 pass / fail 상태를 설정 (특정 텍스트를 체크)
- Route 53 헬스 체커로부터 오는 요청은 라우터/방화벽이 허용하도록 설정해야 함

### Health Checks - Calculated Health Checks

- 여러 개의 헬스 체크 결과를 하나의 헬스 체크로 통합
- **OR, AND, NOT** 조건을 사용 가능
- 최대 256개의 자식 헬스 체크(child health check)를 모니터링할 수 있음
- 부모의 헬스 체크를 통과시키려면 얼마나 많은 자식의 헬스 체크가 필요한지 정의 가능
- e.g., 모든 헬스 체크가 실패하도록 하지 않으면서 웹사이트를 유지보수하고자 할 때

### Health Checks - Private Hosted Zones

- Route 53 헬스체커는 VPC 외부에 존재
- 따라서 **private 엔드포인트**에 접근할 수 없음 (private VPC 또는 온-프로미스 리소스)
- 이에 대처하기 위해, **CloudWatch Metric**을 만들고, 여기에 **CloudWatch Alarm**을 연결 후 해당 알람 자체를 체크하는 헬스 체크를 만들면 됨

## Domain Registar vs. DNS Service

- **Domain Registrar != DNS Service**
- 일반적으로 매년 비용을 청구하며 Domain Registrar로부터 도메인 네임을 구매하거나 등록
- Domain Registrar는 보통 DNS 레코드 관리를 위해 DNS 서비스도 함께 제공하는 경우가 많음
- 하지만, DNS 레코드 관리를 위해 다른 DNS 서비스를 사용할 수도 있음
- e.g.) GoDaddy에서 구매한 도메인을 Route 53에서 DNS 레코드 관리
- 어떻게?
  - Route 53에 Hosted Zone을 생성
  - 써드파티 웹사이트(도메인 제공 업체 ~ ex. GoDaddy)에서 NS 레코드를 Route 53의 **네임 서버**로 변경
