## 애플리케이션 업데이트

```bash
docker build -t getting-started .
docker run -dp 3000:3000 getting-started
```

애플리케이션에 어떤 변경사항이 생겼을 때, 이를 적용하고 이전 챕터에서 했던 것과 동일하게 빌드 / 실행하게되면 아래와 같은 에러가 발생한다.

```bash
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell
(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): Bind for 0.0.0.0:3000 failed: port is already allocated.
```

해당 문제는 컨테이너가 호스트의 3000 포트를 사용하고 있고, 호스트 머신에서는 하나의 프로세스만이 특정 포트를 수신할 수 있기 때문이다. 따라서, 이를 해결하려면 실행 중인 이전의 컨테이너를 제거해야 한다.

## 컨테이너 교체

컨테이너를 제거하려면, 먼저 컨테이너를 멈추어야 한다. 두 가지 방법이 있는데, 어느 쪽을 사용해도 상관없다.

### 1. CLI로 컨테이너 제거

1. `docker ps` 명령으로 컨테이너의 ID를 가져온다.

```bash
docker ps
```

2. `docker stop` 명령으로 컨테이너를 멈춘다.

```bash
# <the-container-id>를 앞선 과정에서 얻은 ID로 교체
docker stop <the-container-id>
```

3. 컨테이너가 멈추고 난 후, `docker rm` 명령으로 제거한다.

```bash
docker rm <the-container-id>
```

만약 "force" 플래그를 추가한다면 `docker rm` 명령 하나만으로 컨테이너를 정지하고 삭제할 수 있다.

```bash
docker rm -f <the-container-id>
```

### 2. Docker 대쉬보드를 통해 컨테이너 삭제

<img src="https://docs.docker.com/get-started/images/dashboard-removing-container.png" />

Docker 대쉬보드를 이용한다면 몇 번의 클릭을 통해 앞선 과정을 해결할 수 있다.

## 컨테이너 재실행

이제, 다시 업데이트된 컨테이너를 실행해보자.

```bash
docker run -dp 3000:3000 getting-started
```
