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
