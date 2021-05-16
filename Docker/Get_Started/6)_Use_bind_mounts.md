이전 챕터에서, DB 내의 데이터를 보존하기 위한 **named volume**에 대해서 이야기했다. Named volume은 어디에 데이터를 저장하는지에 대해서는 신경쓰지 않기 때문에, 단순히 데이터를 저장하기 위한 용도로는 충분하다.

**Bind mounts**를 사용한다면, 우리가 직접 호스트의 정확한 "mountpoint"를 조작할 수 있다. Bind Mounts는 데이터를 보존하기 위해 사용할 수도 있지만, 컨테이너에 추가적인 데이터를 제공해야 하는 경우에 쓰이는 경우가 많다. 애플리케이션 개발 단계에서 Bind Mounts를 사용하면 소스 코드를 컨테이너에 마운트하여 코드 변경 사항을 확인하고, 즉각적인 변경 사항을 확인할 수 있다.

[nodemon](https://npmjs.com/package/nodemon)은 NodeJS 애플리케이션에서 변경 사항을 파악하고, 재실행 시켜주는 툴이다. NodeJS 외의 언어 및 프레임워크에서는 다른 적합한 툴들이 존재할 것이다.

## 볼륨 타입 비교

Bind mounts 와 Named volumes는 Docker 엔진에서 제공되는 두가지 타입의 볼륨이다. 다른 경우에 제공되는 추가적인 볼륨 드라이버를 사용할 수도 있다.

|                                  | Named Volumes               | Bind Mounts                     |
| -------------------------------- | --------------------------- | ------------------------------- |
| 호스트 위치                      | Docker가 정함               | 직접 정함                       |
| 마운트 예시 (`-v` 플래그)        | `my-volume:/usr/local/data` | `/path/to/data:/usr/local/data` |
| 새 볼륨을 컨테이너 컨텐츠로 채움 | 예                          | 아니오                          |
| 볼륨 드라이버 지원               | 예                          | 아니오                          |

## Dev 모드 컨테이너 실행

컨테이너가 개발 워크플로우를 지원하도록 하기 위해서, 아래의 사항을 수행해야 한다.

- 컨테이너에 소스 코드를 마운트시킨다.
- 모든 종속성을 설치한다. (`dev` 종속성 포함)
- `nodemon`을 실행하여 파일시스템 변경을 감시한다.

1. 이전에 실행했던 `getting-started` 컨테이너를 종료, 제거한다.
2. 아래 명령을 입력한다. 아래쪽에서 해당 명령에 대해 상세히 설명하겠다.

```bash
docker run -dp 3000:3000 \
    -w /app -v "$(pwd):/app" \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"
```

PowerShell을 사용한다면 아래 명령을 사용해야 한다.

```shell
docker run -dp 3000:3000 `
    -w /app -v "$(pwd):/app" `
    node:12-alpine `
    sh -c "yarn install && yarn run dev"
```

- `-dp 3000:3000` - 포트 매핑 및 백그라운드 모드 (이전에 언급한 것과 같다.)
- `-w /app` - 작업 디렉토리, 혹은 명령이 실행될 디렉토리를 지정
- `-v "$(pwd):/app"` - 호스트의 현재 디렉토리(`pwd`)를 컨테이너의 `/app` 디렉토리와 **bind mount**시킴
- `node:12-alpine` - 사용할 이미지. Dockerfile 내의 Base 이미지를 사용.
- `sh -c "yarn install && yarn run dev"` - 명령어. `sh`를 통해서 셸을 실행하고(alpine은 `bash`를 갖고있지 않다.) `yarn install`을 실행하여 모든 종속성을 설치한 뒤 `yarn run dev`로 개발 모드로 실행한다.

3. `docker logs -f <container-id>`를 통해 로그를 확인할 수 있다.

```bash
 docker logs -f <container-id>
 $ nodemon src/index.js
 [nodemon] 1.19.2
 [nodemon] to restart at any time, enter `rs`
 [nodemon] watching dir(s): *.*
 [nodemon] starting `node src/index.js`
 Using sqlite database at /etc/todos/todo.db
 Listening on port 3000
```

4. 이제, 애플리케이션에 변경을 적용해보자. `src/static/js/app.js` 파일에서 텍스트를 간단하게 변경해보겠다.

```
 -                         {submitting ? 'Adding...' : 'Add Item'}
 +                         {submitting ? 'Adding...' : 'Add'}
```

5. 브라우저가 변경을 감지하고 페이지를 새로고침하는 것을 확인할 수 있다.

<img src="https://docs.docker.com/get-started/images/updated-add-button.png" />

6. 모든 작업이 끝났다면, 컨테이너를 정지시키고 `docker build -t getting-started .` 명령을 통해 새로운 이미지를 빌드한다.

bind mounts의 이용은 로컬 개발 환경에서 매우 일반적으로 사용된다. 호스트 머신에서 별도로 빌드 툴과 환경을 설치하지 않아도 된다는 장점이 있다. 덕분에 단순히 `docker run` 커맨드를 실행함으로써 개발 환경이 구축되고, 곧바로 개발에 돌입할 수 있다.

추후에 **Docker Compose**에 대해 이야기할텐데, 이를 사용하면 앞서 사용했던 명령들을 훨씬 간단하게 처리할 수 있다.
