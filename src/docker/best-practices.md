## 보안 스캐닝

이미지를 빌드하고 나서, `docker scan` 명령으로 보안 취약점을 탐색하는 것이 좋다. Docker는 [Snyk](http://snyk.io/)과 파트너쉽을 보유하여 취약점 탐색 서비스를 제공한다.

예를 들어, `getting-started` 이미지에 대한 스캐닝을 진행해보자.

```bash
docker scan getting-started
```

스캔은 지속적으로 업데이트되는 취약점 데이터베이스를 활용하기 때문에, 아래와 같이 표시되는 출력은 상황에 따라 달라질 수 있다.

```
✗ Low severity vulnerability found in freetype/freetype
  Description: CVE-2020-15999
  Info: https://snyk.io/vuln/SNYK-ALPINE310-FREETYPE-1019641
  Introduced through: freetype/freetype@2.10.0-r0, gd/libgd@2.2.5-r2
  From: freetype/freetype@2.10.0-r0
  From: gd/libgd@2.2.5-r2 > freetype/freetype@2.10.0-r0
  Fixed in: 2.10.0-r1

✗ Medium severity vulnerability found in libxml2/libxml2
  Description: Out-of-bounds Read
  Info: https://snyk.io/vuln/SNYK-ALPINE310-LIBXML2-674791
  Introduced through: libxml2/libxml2@2.9.9-r3, libxslt/libxslt@1.1.33-r3, nginx-module-xslt/nginx-module-xslt@1.17.9-r1
  From: libxml2/libxml2@2.9.9-r3
  From: libxslt/libxslt@1.1.33-r3 > libxml2/libxml2@2.9.9-r3
  From: nginx-module-xslt/nginx-module-xslt@1.17.9-r1 > libxml2/libxml2@2.9.9-r3
  Fixed in: 2.9.9-r4
```

출력에는 취약점의 타입을 나열하고, 이와 관련된 URL을 보여주며, 취약점을 고치기 위한 최근 라이브러리 버전 등을 제공해준다.

[Docker scan 문서](https://docs.docker.com/engine/scan/)에서 더 많은 옵션들에 대한 사항을 살펴볼 수 있다.

CLI 외에도, Docker Hub 설정을 통해 새롭게 푸쉬된 이미지들에 대해서 스캐닝을 진행할 수도 있다.

<img src="https://docs.docker.com/get-started/images/hvs.png" />

## 이미지 레이어링

한 이미지가 어떻게 구성되어 있는지 확인하려면 어떻게 해야할까? `docker image history` 명령을 사용하면, 하나의 이미지를 구성하기 위해 생성된 여러 레이어들에 대한 명령어들을 확인할 수 있다.

1. `docker image history` 명령을 사용하면 이전 챕터에서 만들었던 `getting-started` 이미지에 대한 레이어들을 확인할 수 있다.

```
docker image history getting-started
```

결과는 아래와 같은 형태일 것이다.

```
 IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
 a78a40cbf866        18 seconds ago      /bin/sh -c #(nop)  CMD ["node" "src/index.j…    0B
 f1d1808565d6        19 seconds ago      /bin/sh -c yarn install --production            85.4MB
 a2c054d14948        36 seconds ago      /bin/sh -c #(nop) COPY dir:5dc710ad87c789593…   198kB
 9577ae713121        37 seconds ago      /bin/sh -c #(nop) WORKDIR /app                  0B
 b95baba1cfdb        13 days ago         /bin/sh -c #(nop)  CMD ["node"]                 0B
 <missing>           13 days ago         /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
 <missing>           13 days ago         /bin/sh -c #(nop) COPY file:238737301d473041…   116B
 <missing>           13 days ago         /bin/sh -c apk add --no-cache --virtual .bui…   5.35MB
 <missing>           13 days ago         /bin/sh -c #(nop)  ENV YARN_VERSION=1.21.1      0B
 <missing>           13 days ago         /bin/sh -c addgroup -g 1000 node     && addu…   74.3MB
 <missing>           13 days ago         /bin/sh -c #(nop)  ENV NODE_VERSION=12.14.1     0B
 <missing>           13 days ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
 <missing>           13 days ago         /bin/sh -c #(nop) ADD file:e69d441d729412d24…   5.59MB
```

각각의 한줄은 이미지 내 하나의 레이어들을 나타낸다. 최근의 레이어일수록 상단에 위치한다. 이를 통해, 각각의 레이어 사이즈를 찾아볼 수 있고, 큰 이미지들을 진단하는데에 도움이 된다.

2. 위에서, 여러 줄들이 결과 출력에서 잘린 것을 볼 수 있는데, `--no-trunc` 플래그를 추가해서 전체 출력을 표시할 수 있다.

```bash
docker image history --no-trunc getting-started
```

## 레이어 캐싱

레이어가 동작하는 방식을 살펴봤으니, 이제 컨테이너 이미지를 빌드하는데에 걸리는 시간을 줄이기 위한 방법에 대해 살펴보자.

**_일단 하나의 레이어가 변경된다면, 해당 레이어의 다운스트림 레이어들도 모두 재생성되어야 한다._**

기존에 작성했던 Dockerfile을 다시 살펴보자.

```
# syntax=docker/dockerfile:1
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

이미지 히스토리에서 살펴본 것처럼, Dockerfile 내의 각각의 명령어들이 이미지 내 레이어가 되는 것을 볼 수 있다. 아마, 이미지에 어떤 변화가 발생하면 모든 yarn 종속성들이 전부 새로 설치되는 것을 확인할 수 있을 것이다. 빌드할 때마다 매번 동일한 종속성들을 설치하는게 적절해보이진 않는다. 이를 보완하려면 어떻게 해야할까?

이를 보완하기 위해서는, Dockerfile이 종속성 캐싱을 지원하도록 수정해야한다. Node 기반의 애플리케이션의 경우, 종속성들은 `package.json` 파일에 정의된다. 만약 우리가 처음 한번만 `package.json` 파일을 복사하고, 종속성을 설치한 후, 그 다음에 다른 것들을 복사하는 형태로 진행한다면 어떨까? 그렇다면 `package.json`에 변경이 있을 때에만 yarn 종속성이 재설치되도록 할 수 있을 것이다.

1. Dockerfile에 가장 먼저 `package.json`을 복사하도록 수정한다. 이후 종속성을 설치하고, 그 다음에 나머지 모두를 복사한다.

```dockerfile
 # syntax=docker/dockerfile:1
 FROM node:12-alpine
 WORKDIR /app
 COPY package.json yarn.lock ./
 RUN yarn install --production
 COPY . .
 CMD ["node", "src/index.js"]
```

2. 동일한 폴더에 `.dockerignore` 라는 이름의 파일을 생성하고 아래의 내용을 작성한다.

```
node_modules
```

`.dockerignore` 파일은 이미지와 관련된 파일만을 선택적으로 복사하기 위한 쉬운 방법이다. [여기](https://docs.docker.com/engine/reference/builder/#dockerignore-file)에서 더 많은 정보를 찾아볼 수 있다. 위 경우에서는, `node_modules` 폴더가 이미지에 추가될 필요가 없으므로 제외시켰다. 어차피 `RUN` 명령 단계를 통해 종속성이 설치되기 때문이다. [여기 문서](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)를 통해 NodeJS 애플리케이션을 도커라이징하는데 있어서 추천되는 디테일 사항에 대해 살펴볼 수 있다.

3. `docker build`를 통해 새로운 이미지를 빌드한다.

```
docker build -t getting-started .
```

그러면 아래와 같은 결과가 나타날 것이다.

```
Sending build context to Docker daemon  219.1kB
Step 1/6 : FROM node:12-alpine
---> b0dc3a5e5e9e
Step 2/6 : WORKDIR /app
---> Using cache
---> 9577ae713121
Step 3/6 : COPY package.json yarn.lock ./
---> bd5306f49fc8
Step 4/6 : RUN yarn install --production
---> Running in d53a06c9e4c2
yarn install v1.17.3
[1/4] Resolving packages...
[2/4] Fetching packages...
info fsevents@1.2.9: The platform "linux" is incompatible with this module.
info "fsevents@1.2.9" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 10.89s.
Removing intermediate container d53a06c9e4c2
---> 4e68fbc2d704
Step 5/6 : COPY . .
---> a239a11f68d8
Step 6/6 : CMD ["node", "src/index.js"]
---> Running in 49999f68df8f
Removing intermediate container 49999f68df8f
---> e709c03bc597
Successfully built e709c03bc597
Successfully tagged getting-started:latest
```

Dockerfile을 수정했으므로 모든 레이어들이 다시 빌드됨을 확인할 수 있다.

4. 이제, `src/static/index.html` 상에서 약간의 변경을 해보자.

5. 다시 `docker build -t getting-started .`를 통해 도커 이미지를 빌드한다. 이번에는 결과가 조금 다르게 출력될 것이다.

```
 Sending build context to Docker daemon  219.1kB
 Step 1/6 : FROM node:12-alpine
 ---> b0dc3a5e5e9e
 Step 2/6 : WORKDIR /app
 ---> Using cache
 ---> 9577ae713121
 Step 3/6 : COPY package.json yarn.lock ./
 ---> Using cache
 ---> bd5306f49fc8
 Step 4/6 : RUN yarn install --production
 ---> Using cache
 ---> 4e68fbc2d704
 Step 5/6 : COPY . .
 ---> cccde25a3d9a
 Step 6/6 : CMD ["node", "src/index.js"]
 ---> Running in 2be75662c150
 Removing intermediate container 2be75662c150
 ---> 458e5c6f080c
 Successfully built 458e5c6f080c
 Successfully tagged getting-started:latest
```

빌드 타임이 훨씬 짧아졌음을 확인할 수 있을 것이다. 그리고 위의 1 ~ 4 단계들이 전부 `Using cache`로 처리된 것을 볼 수 있다.

## 멀티스테이지 빌드

현재 튜토리얼에서 너무 깊은 내용을 다루진 않겠지만, 멀티스테이지 빌드는 한 이미지를 생성하기 위해 여러 스테이지를 사용하도록 도와주는 유용한 툴이다. 이는 다음과 같은 이점을 제공한다.

- 런타임 종속성과 빌드타임 종속성을 분리한다.
- 애플리케이션 구동에 오직 필요한 내용만 빌드하여 전반적인 이미지 사이즈를 줄인다.

### React 예시

React 애플리케이션을 빌드할 때, JS 코드, SASS, 그 외 정적 파일들을 컴파일하기 위한 Node 환경이 필요하다. (특히 JSX) 만약 SSR을 적용하는 것이 아니라면, 프로덕션 빌드 상에는 굳이 Node 환경이 필요하지 않다. 때문에 아래 예시에서는 정적인 nginx 컨테이너에 정적 리소스들만을 복사해준다.

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

위에서는 `node:12` 이미지를 사용해 빌드를 수행한 후, nginx 컨테이너에 그 결과를 복사해넣게끔 해주었다.
