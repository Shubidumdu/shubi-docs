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
