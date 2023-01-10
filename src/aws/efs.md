# EFS

- 여러 EC2 인스턴스에 마운트 될 수 있는 관리형 NFS (Managed NFS ~ network file system)
- EFS는 여러 AZ에 있는 EC2 인스턴스와 함께 동작할 수 있음
- 높은 가용성이 있고(highly available), 확장 가능하며(scalable), 비쌈(expensive ~ gp2의 3배), 사용한 만큼 지불 (pay per use)
- 사례 :
  - content management
  - web serving
  - data sharing
  - wordpress
- NFSv4.I 프로토콜을 사용
- EFS에 대한 엑세스를 관리하기 위해 Security Group을 사용
- **Linux based AMI의 경우에만 호환됨 (Windows는 안됨)**
- KMS를 이용해 암호화 (encryption at rest using KMS)
- 표준 파일 API를 가진 POSIX 파일 시스템 (~LINUX)
- 파일 시스템은 자동으로 크기가 변하며, 사용량에 따라 비용이 청구됨. 별도로 capacity plan이 없음.

## EFS - Performance & Storage Classes

- EFS Scale
  - 동시에 1000 개의 NFS client를 가질 수 있으며, 10GB/s 이상의 throughput
  - 페타바이트(Petabyte)급의 네트워크 파일 시스템으로 자동으로 확장될 수 있음

- Performance mode (EFS 생성 시에 설정됨)
  - General purpose (default): latency에 민감한 사례에 사용 (웹서버, CMS, 등등..)
  - Max I/O - 더 높은 latency을 갖춘 반면, 더 높은 throughput과 더 높은 병렬 처리(parallel) ~ 빅데이터, 미디어 프로세싱 사례에 사용

- Throughput mode
  - Bursting (1TB = 50MB/s + burst of up to 100MB/s)
  - Provisioned (= Enhanced로 명칭 변경): 스토리지 사이즈와 무관하게 throughput을 설정 (Ex. 1TB 스토리지에 1GB/s으로 설정)

- Storage Tiers (lifecycle management feature - N일 이후에 파일 이동)
  - Standard: 자주 엑세스되는 파일들에 사용
  - Infrequent access (EFS-IA): 파일 검색에 비용 부과하며, 저장에는 더 낮은 비용. Lifecycle Policy로 EFS-IA를 활성화.

- Availability and Durability (= Storage class로 명칭 변경)
  - Standard: Multi-AZ, 프로덕션 환경에서 유용
  - One Zone: One AZ, 개발 환경에서 유용, 기본적으로 백업이 활성화 되어 있으며, IA(Infrequent Access)와 호환. (EFS One Zone-IA)

- 최대 90%까지 비용 절감 가능
