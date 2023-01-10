# AMI

- AMI = Amazon Machine Image
- AMI는 EC2 인스턴스의 customization다.
  - 나만의 소프트웨어, 설정, OS, 모니터링 등을 추가할 수 있음
  - 모든 내 소프트웨어가 pre-package 되어 있으므로, 더 빠른 부트/설정 시간을 가질 수 있다.

- AMI는 **하나의 리전**에 대해서만 만들어질 수 있으며, 다른 리전에서 사용하고자 한다면 별도로 복사를 해야한다.
- EC2 인스턴스를 다음과 같은 AMI들에서 실행할 수 있다.
  - 공용(Public) AMI: AWS가 제공
  - 직접 만든 AMI: 직접 만들고 유지보수
  - AWS Marketplace AMI: 다른 누군가가 만든 AMI (판매도 할 수 있음)

## AMI Process (from an EC2 instance)

- EC2 인스턴스를 시작하여 커스터마이징함
- 인스턴스를 중지 (데이터 무결성 ~ data integrity를 위하여)
- AMI를 빌드 - 해당 작업은 EBS 스냅샷도 마찬가지로 생성함
- 다른 AMI들로부터 인스턴스를 실행
