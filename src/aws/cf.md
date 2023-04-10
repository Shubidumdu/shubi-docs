# CloudFront

- Content Delivery Network (CDN) - 컨텐츠 전송 네트워크
- **컨텐츠를 엣지에 캐시하여 읽기 성능을 향상**
- 이용자 경험 향상
- 전 세계 216개 지점 (엣지 로케이션)
- **DDoS 보호 (전 세계적으로 애플리케이션이 배포되기 때문), AWS Shield, AWS Web Application Firewall와 통합**

## CloudFront - Origins

![AWS CloudFront](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/images/cloudfront-events-that-trigger-lambda-functions.png)

- **S3 버킷**
  - 엣지에 파일을 배포하고 캐시하는 용도로 사용
  - CloudFront **Origin Access Control(OAC)**를 사용하여 보안 강화
  - OAC는 Origin Access Indentitiy (OAI)의 대체
  - CloudFront는 S3에 파일을 업로드하기 위한 ingress로 사용될 수 있음

- **Custom Origin (HTTP)**
  - Application Load Balancer
  - EC2 Instance
  - S3 웹사이트 (먼저 반드시 버킷을 정적 S3 웹사이트로 활성화해야함)
  - 원하는 HTTP 백엔드 뭐든지

## CloudFront vs Cross Region Replication

- CloudFront
  - 글로벌 엣지 네트워크
  - TTL 기반으로 파일이 캐시됨 (아마 1일)
  - **어디서든 사용 가능해야 하는 정적 컨텐츠에 유용함**
- S3 Cross Region Replication
  - 복제가 일어나길 원하는 각 리전마다 직접 세팅해주어야 함
  - 거의 실시간으로 파일이 업데이트됨
  - 읽기 전용
  - **일부 지역에서 낮은 레이턴시로 사용해야 하는 동적 컨텐츠에 유용함**

## CloudFront - ALB or EC2 as an origin

![ALB and EC2 as an origin](https://s2.51cto.com/images/blog/202107/30/9919472bfa0c101d1597350721fa406e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

## CloudFront - Geo Restriction

- 내 배포에 액세스할 수 있는 대상을 제한할 수 있음
  - **Allowlist**: 허용한 나라 목록에 포함된 이용자들만 컨텐츠에 액세스 가능
  - **Blocklist**: 금지된 나라 목록에 포함되지 않은 이용자들만 컨텐츠에 액세스 가능
- 대상 국가의 경우 써드파티 Geo-IP 데이터베이스를 통해 결정됨
- 사례: 컨텐츠 액세스를 제어하는 저작권법

## CloudFront - Pricing

- CloudFront 엣지 로케이션은 전 세계에 분포
- 각 엣지 로케이션 마다 데이터 비용이 다름

### CloudFront - Price Classes

![Price Classes Map](https://user-images.githubusercontent.com/29729545/153042484-2e8e09f3-92ad-494d-ad07-f79914c14772.png)

- **비용 감축**을 위해 엣지 로케이션의 갯수를 줄일 수 있음
- 3개의 Price class가 존재
  - Price Class All: 모든 리전 - 최고 성능
  - Price Class 200: 대부분의 리전, 제일 비싼 리전들은 제외
  - Price Class 100: 가장 저렴한 리전만

## CloudFront - Cache Invalidations

- 내가 백엔드 오리진을 업데이트했을 경우, CloudFront는 그것을 알 방도가 없기 때문에, 오직 TTL이 만료되고 나서야 갱신된 컨텐츠를 제공하게 됨
- 다만, **CloudFront Invalidation**을 통해서 강제로 전체 또는 일부 캐시를 갱신(refresh)할 수 있음 (TTL을 우회)
- 모든 파일(`*`), 또는 특정 경로(`/images/*`)에 대해 무효화 할 수 있음

## Global Accelerator

- 배포된 애플리케이션에 직접 접근하고자 하는 글로벌 이용자들은, 많은 홉(hop)으로 인해 레이턴시가 길어질 수 있는 공용 인터넷을 통해 전송됨.
- 이 때, 레이턴시를 최소화하기 위해 AWS 네트워크를 거쳐 최대한 빠르게 이동하고자 함

### Unicast IP vs Anycast IP

- **Unicast IP**: 하나의 서버는 하나의 IP를 가짐
- **Anycast IP**: 모든 서버가 동일한 IP를 가지며, 클라이언트는 가장 가까운 서버로 라우트됨

### Global Accelerator - Overview

![Global Accelerator](https://d1.awsstatic.com/product-page-diagram_AWS-Global-Accelerator%402x.dd86ff5885ab5035037ad065d54120f8c44183fa.png)

- AWS 내부 네트워크를 활용하여 애플리케이션으로 라우팅
- 애플리케이션에 대한 **2개의 Anycast IP**를 생성
- Anycast IP는 엣지 로케이션으로 트래픽을 직접 전송
- 엣지 로케이션은 애플리케이션으로 트래픽을 전송

### Global Accelerator - Description

- **퍼블릭/프라이빗 상관없이 Elastic IP, EC2 인스턴스, ALB, NLB**과 함께 사용할 수 있음
- 일관된 성능
  - 지능적인 라우팅을 통한 낮은 레이턴시와 빠른 리전 내 장애 복구
  - 클라이언트 캐시와 관련한 문제 없음
  - 내부적으로 AWS 네트워크 사용
- 헬스 체크
  - Global Accelerator는 애플리케이션에 대한 헬스체크를 수행함
  - 애플리케이션이 글로벌화 될 수 있도록 도와줌 (unhealthy 상태인 경우 1분 이하로 장애 복구)
  - 이러한 헬스체크 덕에 재해 복구(disaster recovery)에 유용함
- 보안
  - 오직 2개의 외부 IP만 화이트리스트에 추가하면 됨
  - AWS Shield 덕에 DDoS 보호

### Global Accelerator vs CloudFront

- 둘다 AWS 글로벌 네트워크과 그것의 엣지 로케이션을 사용함
- 두 서비스 모두 AWS Shield를 통한 DDoS 보호를 할 수 있음

- **CloudFront**
  - 캐시 가능한 컨텐츠 모두에 대한 성능 향상 (이미지나 비디오)
  - 동적 컨텐츠 (API 가속 및 동적 사이트 전송)
  - 컨텐츠가 엣지를 통해 제공됨
- **Global Accelerator**
  - TCP나 UDP를 거치는 넓은 범위의 애플리케이션
  - 하나 이상의 AWS 리전에 실행되는 애플리케이션으로 엣지에서의 패킷을 프록시해줌
  - non-HTTP 사례에 유용함 - 게이밍 (UDP), IoT (MQTT), 또는 Voice over IP(VoIP)
  - 정적인 IP 주소들을 필요로 하는 사례에 적합함

  - 결정론적(deterministic)이고 빠른 지역 장애 복구가 필요한 사례에 적합함
