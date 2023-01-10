# EBS

- **EBS (Elastic Block Store) 볼륨**은 인스턴스가 돌아가는 동안에 부착할 수 있는 **네트워크 드라이브**이다.
- 인스턴스가 종료된 이후에도 데이터를 보존할 수 있도록 해준다.
- 기본적으로는 한번에 하나의 인스턴스에만 이용될 수 있으나, 일부 EBS에는 **multi-attach** 기능이 있다.
- 특정 AZ에 격리된다.
- 하나의 네트워크 USB 스틱으로 생각해도 좋다.
- 프리티어의 경우 General Purpose (SSD) 또는 Magenetic으로 한달에 30GB의 무료 EBS 스토리지를 제공받는다.

## EBS Volume

- 네트워크 드라이브에 해당 (즉, 물리적(physical) 드라이브가 아님.)
  - 인스턴스와 소통하기 위해 네트워크를 사용하며, 이는 즉 약간의 latency가 발생할 수 있음을 의미한다.
  - 하나의 EC2 인스턴스로부터 분리(detach)되어, 다른 것에 부착(attach)될 수 있다.

- 하나의 AZ에 갇혀있다.
  - 예를 들어, us-east-1a의 EBS볼륨은 us-east-1b에 부착될 수 없다.
  - 볼륨을 AZ 너머로 이동시키려면, 먼저 그것에 대한 스냅샷(snapshot)을 사용해야 한다.

- 프로비전(provision)된 가용량(capacity)을 갖고 있다. (size in GBs, and IOPS)
  - 모든 프로비전 가용량에는 비용이 지불된다.
  - 추후에도 드라이브 용량을 증가시킬 수 있다.

## Delete on Termination attribute

- EC2 인스턴스가 종료될 때 EBS의 동작을 컨트롤할 수 있다.
  - 기본적으로, 루트 EBS 볼륨(root EBS volume)은 삭제된다. (= enabled)
  - 기본적으로, 그 외의 EBS 볼륨들은 삭제되지 않는다. (= disabled)
- 이는 AWS console / AWS CLI를 통해 이루어질 수 있으며, **인스턴스가 종료(terminate)되더라도 루트 볼륨을 보존하고자 할 때 사용할 수 있다.**

## Snapshots

- 특정한 시점에 보유한 EBS 볼륨에 대한 백업(=스냅샷)을 만들 수 있음
- 스냅샷을 위해 볼륨을 분리(detach)할 필요는 없지만, **권장**되는 사항임.
- 이를 통해 다른 AZ 또는 지역 너머로 스냅샷을 복사하여 볼륨을 복구(restore)할 수 있음.
  
### EBS Snapshot Features

- EBS Snapshot Archive
  - 스냅샷을 75% 싼 스토리지 티어인 archive tier로 이동함.
  - 아카이브한 내용을 복구하는데에 24 ~ 72 시간이 소요됨.

- Recycle Bin for EBS Snapshots
  - 삭제된 스냅샷을 보존하기 위한 것으로, 일시적인 삭제 이후에도 이를 복구할 수 있도록 Rule을 생성할 수 있음
  - 보존기간을 명시할 수 있음 (1일부터 1년까지)

- Fast Snapshot Restore (FSR)
  - 첫 사용에도 latency가 존재하지 않도록 스냅샷의 완전한 초기화를 강제함.

## EBS Volume Types

- EBS 볼륨은 6가지 종류가 있음
  - gp2 / gp3 (SSD): 폭넓은 종류의 작업을 처리할 수 있는 가격과 성능 간 밸런스를 갖춘 일반적인 목적의 SSD
  - io1 / io2 (SSD): 낮은 latency와 높은 throughput이 요구되는 상황에서 사용하는 높은 성능의 SSD
  - st1 (HDD): 자주 액세스 해야하고, throughput이 중요한 작업에 사용하기 위한 용도의 낮은 비용의 HDD
  - sc (HDD): 비교적 액세스 빈도가 낮은 작업을 처리하기 위한 용도의 가장 저렴한 HDD

- EBS 볼륨은 사이즈 / throughput / IOPS (I/O Ops Per Sec)에 따라 특징이 나뉨.
- 헷갈린다면 AWS 문서를 참조할 것.
- **부트 볼륨(root OS가 실행되는 볼륨)으로는 오직 gp2/gp3와 io1/io2만 사용할 수 있음.**

### EBS Volume Types - General Purpose SSD

- 비용 효율적인 스토리지와 낮은 latency가 요구되는 상황에 사용
- Ex.) System boot volumes, Virtual desktops, Development and test environments
- 1GB ~ 16TB
- gp3
  - 기본 3,000 IOPS와 125MB/s의 throughput
  - 각각 16,000 IOPS 및 1000MB/s throughput 까지 높일 수 있음
- gp2
  - 3,000 IOPS 까지 높일 수 있는 작은 gp2 볼륨
  - 볼륨의 사이즈에 따라 IOPS도 달라지며, 최대 IOPS는 16,000
  - GB 당 3 IOPS, 즉 5,334GB에서 최대 IOPS에 도달할 수 있음.

### EBS Volume Types - Provisioned IOPS (PIOPS) SSD

- IOPS 성능을 유지해야 하는 중요한 비즈니스 애플리케이션에 사용
- 또는 16,000 IOPS 이상을 필요로 하는 애플레케이션에 사용
- 데이터베이스 작업에 유용 (스토리지 성능 및 consistency가 중요하기 때문)
- io1/io2 (4GB - 16TB)
  - 최대 PIOPS: Nitro EC2 인스턴스의 경우 64,000 & 그 외에는 32,000
  - 스토리지 사이즈와 별개로 PIOPS를 높일 수 있음
  - io2가 더 높은 내구성(durability)를 갖고 있으며, 각 GB 당 더 많은 IOPS 성능을 가짐 (io1와 동일한 비용인 경우)
- io2 Block Express (4GB - 64TB)
  - ms(밀리초) 미만의 latency가 요구되는 경우
  - 최대 PIOPS: 1,000:1의 IOPS:GB 비율을 가지며, 최대 256,000
- EBS Multi-attach를 지원

### EBS Volume Types - Hard Disk Drives (HDD)

- 부트 볼륨으로 사용될 수 없음
- 125MB - 16TB
- Throughput Optimized HDD (st1)
  - Big Data, Data Warehouses, Log Processing
  - 최대 throuput 500MB/s - 최대 IOPS 500
- Cold HDD (sc1)
  - 액세스 빈도가 낮은 데이터
  - 비용을 낮추는 것이 중요한 상황에서 사용
  - 최대 throughput 250MB/s - 최대 IOPS 250

## EBS Multi-Attach - io1/io2 family

- 동일한 AZ 내에 여러 EC2 인스턴스에 동일한 EBS 볼륨을 부착하는 것.
- 각각의 인스턴스는 고성능 볼륨에 모든 읽기/쓰기 권한을 갖게 됨
- 사례 :
  - Clustered linux application 내에서 높은 애플리케이션 가용성(higher application availability)을 보존해야 할 때
  - 단, 애플리케이션 자체가 동시 쓰기 작업을 다룰 수 있어야 함
- **한꺼번에 최대 16개의 인스턴스에만 적용할 수 있음**
- 클러스터에 대해 인지하는(cluster-aware) 파일 시스템을 사용해야 함. (즉, XFS, EX4 등을 사용할 수 없음)

## EBS Encryption

- 암호화된 EBS 볼륨을 생성하면
  - 볼륨 내에 저장된 데이터가 암호화됨
  - 인스턴스와 볼륨 사이를 이동하는 모든 데이터가 암호화됨
  - 모든 스냅샷이 암호화됨
  - 스냅샷으로부터 생성되는 모든 볼륨이 암호화됨
- 암호화/복호화(encryption/decryption)는 투명하게 처리됨 (따로 처리해야 할 일이 없음)
- 암호화는 latency에 거의 영향을 끼치지 않음
- EBS 암호화는 KMS(AES-256) 암호화를 사용
- 암호화되지 않은 스냅샷을 복사하는 경우 암호화가 가능
- 암호화된 볼륨의 스냅샷은 암호화됨

### Encryption: encrypt an unencrypted EBS Volume

- 볼륨의 EBS 스냅샷을 생성
- EBS 스냅샷을 암호화 (using copy)
- 스냅샷을 통해 새 EBS 볼륨을 생성 (새로 생성된 볼륨은 암호화됨)
- 기존 인스턴스에 암호화된 볼륨을 부착
