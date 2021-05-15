## 시작하기

```bash
docker run -d -p 80:80 docker/getting-started
```

- `-d` : 컨테이너를 `detached` 모드로 실행한다. (백그라운드에서)
- `-p 80:80` : 호스트의 `80` 포트를 컨테이너의 `80` 포트로 연결시킨다.
- `docker/getting-started` : 사용할 이미지

위의 단일 문자 플래그들은 합쳐서 사용할 수 있다. 이를테면 아래와 같이 작성할 수 있다.

```bash
docker run -dp 80:80 docker/getting-started
```

## 대쉬보드

컨테이너를 실행하고나면 이를 대쉬보드 상에서 확인할 수 있다.

<img src="https://docs.docker.com/get-started/images/tutorial-in-dashboard.png" />

## 컨테이너란?

컨테이너란 호스트 머신의 다른 프로세스로부터 격리된 또 하나의 프로세스다. 이러한 분리는 Linux에서 오랫동안 사용되어 온 기능인 [kernel namespaces와 cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504)를 활용한다. 그리고 Docker는 이런 기능들은 접근 가능하고 사용하기 쉽게 만들고자 한 것이다.

## 컨테이너 이미지란?

컨테이너가 실행될 때, 해당 컨테이너는 격리된 파일시스템을 사용한다. 이 격리된 파일시스템이 바로 **컨테이너 이미지**로부터 제공된다. 각 이미지들은 컨테이너의 파일시스템을 내포하고 있으며, 애플리케이션의 실행에 필요한 모든 것들을 담고 있어야 한다. (dependencies / configurations / scripts / binaries / etc.) 또한 환경변수, 기초 실행 명령 / 그 외의 메타 데이터 등 컨테이너에 대한 다른 설정들 또한 갖고있다.

이미지에 대해서는 Layering, Best practices 등 추후 더 깊게 다루어보도록 하겠다.
