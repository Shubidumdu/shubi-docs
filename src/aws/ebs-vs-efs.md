# EBS vs. EFS

## EBS Volumes

- 특징
  - 기본적으로는 하나의 인스턴스에 하나씩 부착
  - AZ 레벨에서 격리됨
  - gp2: 디스크 사이즈가 늘어남에 따라 IO 성능도 늘어남
  - io1: 디스크 사이즈와 무관하게 IO 성능을 별개로 늘릴 수 있음

- EBS 볼륨을 AZ 너머로 마이그레이션하려면
  - 스냅샷을 찍고
  - 다른 AZ에서 해당 스냅샷을 복구
  - EBS 백업은 IO를 사용하며, 현재 애플리케이션이 많은 트래픽을 다루는 중인 경우에는 이를 작동시키지 말아야 한다.

- EC2 인스턴스가 종료되면, 기본적으로 해당 인스턴스의 EBS 볼륨도 종료된다. (해당 동작은 disable할 수 있음.)

## EFS - Elastic File System

- 특징
  - AZ와 무관하게 100개 까지 인스턴스를 마운팅 할 수 있음
  - 웹사이트 파일 공유에 활용할 수 있음 (Ex. WordPress)
  - 리눅스 인스턴스(POSIX)에만 사용할 수 있음
  - EBS보다 비용이 더 높지만, 비용 절감을 위해 EFS-IA를 활성화할 수 있음

### EC2 Instance Store

- 극도로 높은 I/O 성능이 요구되는 상황에서 사용
- 기본적으로 인스턴스가 중지되면 데이터가 사라짐 (저장이 일시적임)
  