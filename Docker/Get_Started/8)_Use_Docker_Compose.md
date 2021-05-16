[Docker Compose](https://docs.docker.com/compose/)는 멀티 컨테이너 애플리케이션을 정의하고 공유하는 것을 돕고자 개발된 툴이다. Compose를 사용하면, YAML 파일을 생성하여 서비스를 정의한 후, 단일 명령을 통해 애플리케이션을 구축하거나 분해할 수 있다.

Compose를 사용할 경우의 큰 이점은 바로 애플리케이션 스택을 하나의 파일로 정의할 수 있다는 점이다. 해당 파일은 프로젝트 저장소의 루트에 저장되며, 다른 이들이 이를 보고 프로젝트에 쉽게 기여할 수 있게 해준다. 그런 경우가 아니더라도, 단순히 repo를 클론하여 애플리케이션을 실행하기만 하면 된다는 점에서 편리하다. 이는 Github/GitLab 등에서 이루어지는 것과 유사하다.

## Docker Compose 설치

Windows나 Mac에서 Docker Desktop/Toolbox를 설치했다면, 이미 Docker Compose가 포함되어 있다. "Play-with-Docker" 인스턴스 역시 Docker Compose가 설치되어 있으며, Linux 머신을 사용하는 상황이라면, 별도로 [설치](https://docs.docker.com/compose/install/)해주어야 한다.

설치 이후, 아래 명령을 통해 버전 정보를 확인해보자.

```
docker-compose version
```

## Compose 파일 생성

1. 프로젝트 루트에서, `docker-compose.yml` 이라는 이름의 파일을 생성한다.

2. 해당 파일에서, 우선 schema 버전을 정의하는 것부터 시작하자. 대부분의 경우에는 최신 지원 버전을 사용하는 것이 좋다. 현재 schema 버전에서 가능한 Compose file의 [레퍼런스](https://docs.docker.com/compose/compose-file/)에 대해 확인하자.

```yaml
version: '3.7'
```

3. 이후, 애플리케이션에 사용할 각각의 서비스(컨테이너)를 정의해준다.

```yaml
version: '3.7'

services:
```

## 앱 서비스 정의

이전 챕터에서 우리는 앱 컨테이너를 정의하기 위해 아래와 같은 명령을 실행했다.

```bash
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

PowerShell의 경우

```sh
docker run -dp 3000:3000 `
  -w /app -v "$(pwd):/app" `
  --network todo-app `
  -e MYSQL_HOST=mysql `
  -e MYSQL_USER=root `
  -e MYSQL_PASSWORD=secret `
  -e MYSQL_DB=todos `
  node:12-alpine `
  sh -c "yarn install && yarn run dev"
```

1. 먼저, 컨테이너에 사용할 서비스 엔트리와 이미지를 정의한다. 서비스에는 어떤 이름이든 쓰여도 된다(여기에서는 `app`). 여기서의 이름은 자동으로 network alias가 되며, 이후 다른 서비스에서 해당 서비스를 활용할 때 유용하다.

```yaml
version: '3.7'

services:
  app:
    image: node:12-alpine
```

2. 일반적으로 `command` 항목을 `image` 가까이에 정의하긴 하지만, 별도로 순서에 대한 요구 사항은 없다.

```yaml
version: '3.7'

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
```

3. 서비스의 `ports`를 정의해준다. (`-p 3000:3000`) 이 부분에서는 [짧게](https://docs.docker.com/compose/compose-file/compose-file-v3/#short-syntax-1) 작성할 수도 있고, 좀 더 [길게](https://docs.docker.com/compose/compose-file/compose-file-v3/#long-syntax-1) 구체적으로 작성할 수도 있다.

```yaml
version: '3.7'

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
```

4. 이제 `working_dir`과 `volumes` 항목을 통해 작업 디렉토리(`-w /app`)와 볼륨 매핑(`-v "$(pwd):/app"`)를 정의한다. 볼륨 역시 [짧게](https://docs.docker.com/compose/compose-file/compose-file-v3/#short-syntax-3) 혹은 [길게](https://docs.docker.com/compose/compose-file/compose-file-v3/#long-syntax-3) 작성될 수 있다.

Docker Compose를 통해 볼륨을 정의하는 경우, 현재 디렉토리에서의 상대 경로를 사용할 수 있다는 장점이 있다.

```yaml
version: '3.7'

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
```

5. 마지막으로, 사용하는 환경 변수를 `environment` 항목에서 지정해준다.

```yaml
version: '3.7'

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

## MySQL 서비스 정의

자, 이제 MySQL 서비스에 대해 정의해보자. 앞서 아래와 같은 명령을 통해 MySQL 컨테이너를 실행했다.

```bash
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

PowerShell을 사용한다면

```sh
docker run -d `
  --network todo-app --network-alias mysql `
  -v todo-mysql-data:/var/lib/mysql `
  -e MYSQL_ROOT_PASSWORD=secret `
  -e MYSQL_DATABASE=todos `
  mysql:5.7
```

1. 먼저 새로운 서비스를 정의하고 `mysql`로 이름짓는다. 이를 통해 자동으로 network alias를 갖게 된다.

```yaml
version: '3.7'

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
```

2. 다음으로, 볼륨 매핑을 정의한다. 기존에는 `docker run`을 통해 컨테이너를 실행하게 되면, named volume이 알아서 생성되었지만, Compose를 사용하는 경우에는 그렇지 않다. 최상단의 `volumes:` 항목에서 직접 볼륨을 정의하고, `services:` 의 각 항목 내에서 mountpoint를 지정해주어야 한다. 단순히 볼륨 네임만을 정의한다면 기본 옵션값들이 사용된다. 추가 옵션들에 대해서는 [여기](https://docs.docker.com/compose/compose-file/compose-file-v3/#volume-configuration-reference)를 참조하자.

```yaml
version: '3.7'

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql

volumes:
  todo-mysql-data:
```

3. 마지막으로, 환경 변수들을 정의해준다.

```yaml
version: '3.7'

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

최종적으로, 작성한 `docker-compose.yml`파일은 아래와 같은 형태가 된다.

```yaml
version: '3.7'

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

## 애플리케이션 스택 실행

`docker-compose.yml` 파일을 갖고 있다면, 이제 애플리케이션들을 실행할 수 있다.

1. 기존에 실행 중인 app 혹은 db가 없는지 확인한다. (`docker ps`와 `docker rm -f <ids>` 사용)

2. `docker-compose up`을 통해 애플리케이션 스택을 실행한다. `-d` 플래그를 통해 모든 스택들을 백그라운드 모드에서 실행한다.

```bash
docker-compose up -d
```

명령 이후 아래와 같은 진행사항들이 출력된다.

```bash
Creating network "app_default" with the default driver
Creating volume "app_todo-mysql-data" with default driver
Creating app_app_1   ... done
Creating app_mysql_1 ... done
```

네트워크와 볼륨들이 생성되는 것을 확인할 수 있다. 기본적으로, Docker Compose는 애플리케이션 스택 상에 정의된 네트워크를 생성한다. (별도로 compose 파일 내에서 네트워크를 정의하지 않은 이유다.)

3. `docker-compose logs -f` 명령을 통해 로그를 확인할 수 있다. 각각의 서비스에서의 로그가 단일 스트림으로 인터리빙된것을 볼 수 있다. 이는 타이밍 관련 이슈들을 탐지하고자 하는 경우에 굉장히 유용하다. `-f` 플래그는 로그를 "팔로우"할 것을 의미하며, 실시간으로 재생되는 로그들을 제공받는다.

```
mysql_1  | 2019-10-03T03:07:16.083639Z 0 [Note] mysqld: ready for connections.
mysql_1  | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
app_1    | Connected to mysql db at host mysql
app_1    | Listening on port 3000
```

서비스 네임이 각 줄의 `|` 좌측에 출력되며, 만약 특정 서비스에 대한 로그만을 확인하고자 한다면, 로그 명령 끝에 원하는 서비스네임을 추가해주면 된다.

```
docker-compose logs -f app
```

> **DB 실행 이후 App이 실행되도록 하기** : Docker는 다른 컴포넌트가 완전히 구동되고 준비될 때까지 기다리도록 하는 내장 지원 기능이 별도로 없다. Node 기반의 프로젝트에서는 이를 위해 [wait-port](https://github.com/dwmkerr/wait-port) 라이브러리를 사용할 수 있다. 다른 프레임워크 및 언어에 대해서도 유사한 것들이 존재한다.

4. 이제 애플리케이션을 직접 열어 확인해보자.

## 대쉬보드에서 애플리케이션 스택 확인

Docker 대쉬보드를 살펴보면 `app`이라는 이름으로 그룹이 생성되었음을 확인할 수 있다. 이는 Docker Compose에서 작성된 프로젝트 네임으로, 기본값으로 `docker-compose.yml`이 위치한 디렉토리의 이름이 사용된다.

<img src="https://docs.docker.com/get-started/images/dashboard-app-project-collapsed.png" />

애플리케이션 스택을 살펴보면, 앞서 compose 파일을 통해 정의한 두 컨테이너를 확인할 수 있다. 이들은 `<project-name>_<service-name>_<replica-number>`의 패턴을 따른다.

<img src="https://docs.docker.com/get-started/images/dashboard-app-project-expanded.png" />

## 종료 및 제거

`docker-compose down`을 명령하거나 대쉬보드 상에서 휴지통 아이콘을 클릭하면 앱 스택을 종료할 수 있다. 관련된 모든 컨테이너들이 중지되고, 네트워크는 삭제된다.

> **주의** : 기본적으로 compose 파일에서 정의된 named volume들은 `docker-compose down` 명령으로 제거되지 않는다. 볼륨들도 제거하고자 하는 경우, `--volumes` 플래그를 추가해주어야 한다.
>
> Docker 대쉬보드에서도 마찬가지로 애플리케이션 스택을 제거할 때 볼륨은 제거하지 않는다.

앱 스택을 제거 한 이후, 다른 프로젝트를 작업할 때도 단순히 `docker-compose up`를 실행하기만 하면 된다.
