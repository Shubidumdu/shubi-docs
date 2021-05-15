## 애플리케이션 구성

<img src="https://docs.docker.com/get-started/images/ide-screenshot.png" />

위와 같은 애플리케이션이 있다고 하자.

## 컨테이너 이미지 빌드

애플리케이션을 빌드하기 위해서는 `Dockerfile`을 사용해야 한다. `Dockerfile`은 컨테이너 이미지를 생성하기 위해 사용되는 간단한 텍스트 스크립트다.

1. 먼저 `Dockerfile`을 `package.json`이 위치한 폴더와 같은 곳에 생성한다.

```dockerfile
 # syntax=docker/dockerfile:1
 FROM node:12-alpine
 RUN apk add --no-cache python g++ make
 WORKDIR /app
 COPY . .
 RUN yarn install --production
 CMD ["node", "src/index.js"]
```

`Dockerfile`에는 별도로 `.txt`와 같은 확장자가 붙어있지 않음을 유의하자.

2. `Dockerfile`이 위치한 디렉토리로 이동하여 `docker build` 명령을 통해 컨테이너 이미지를 빌드한다.

```bash
docker build -t getting-started .
```

빌드 과정에서, 수많은 **layer**들이 다운로드되는 것을 확인할 수 있는데, 이는 `Dockerfile`의 처음에 `node:12-alpine` 이미지에서부터 시작된다고 빌더에게 명령했기 때문이다. 현재의 호스트 머신에는 이 이미지가 존재하지 않고, 따라서 해당 이미지를 다운로드하는 과정이 필요한 것이다.

해당 이미지가 다운로드되면, 애플리케이션을 복사하고 `yarn`으로 애플리케이션의 dependencies를 설치한다. `CMD` 명령에는 이미지로부터 컨테이너를 가동할 때 실행할 기본 명령어를 지정한다.

마지막으로, `-t` 플래그는 이미지에 대한 태그를 의미한다. 생성한 이미지에 대한 읽기 쉬운 이름이라고 이해하면 된다. `getting-started` 라는 이름으로 이미지를 이름지었기 때문에, 컨테이너를 실행할 때마다 해당 이름을 참조할 수 있다.

`docker build`의 마지막에 있는 `.`은 Docker에게 **현재 디렉토리**에서 `Dockerfile`을 찾아야한다고 명령하는 것이다.

## 앱 컨테이너 실행

자, 이제 이미지를 만들었으니, 이를 실행해보자. `docker run` 명령을 사용하면 된다.

1. 앞서 만든 이미지를 `docker run` 명령을 통해 컨테이너로 실행한다.

```bash
docker run -dp 3000:3000 getting-started
```

앞선 챕터에서 `-d`와 `-p` 플래그에 대해 설명했던 것이 기억나는가? `-dp` 플래그를 통해, 호스트의 3000 포트를 컨테이너의 3000 포트와 매핑하고, 컨테이너를 "detached" 모드(백그라운드에서) 실행했다. 만약 별도로 포트를 지정해주지 않는다면, 애플리케이션에 접근할 수 없다.

2. 잠시 후, `http://localhost:3000`에 접근하면, 애플리케이션을 확인할 수 있다.
