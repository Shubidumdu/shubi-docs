이전 챕터까지는 모두 단일 컨테이너 애플리케이션을 다루었다. 자, 이제 MySQL과 같이 별도의 애플리케이션 스택을 추가하려고 한다. 종종 다음과 같은 물음이 생긴다. - "어디서 MySQL을 구동해야 할까? 똑같은 컨테이너 내에서 설치되어야 하나, 아니면 따로 실행되어야 하나?" 일반적으로, **각각의 컨테이너는 한 가지 일만을 수행하고, 그것이 잘 이루어져야 한다.** 이유는 다음과 같다.

- DB 외에 API 및 프론트엔드를 확장해야 할 가능성이 높다.
- 별도의 컨테이너를 통한 버저닝이 수월하다.
- 로컬에서 데이터베이스에 대한 컨테이너를 사용할 수도 있는 한편, 프로덕션 상에 운영 중인 데이터베이스에 대해 관리되는 서비스를 사용하고 싶을수도 있다.
- 여러 프로세스를 실행하려면 별도의 프로세스 매니저가 요구된다. (컨테이너는 하나의 프로세스만을 실행한다.) 따라서 컨테이너의 시작 / 종료에 복잡성을 가중시킨다.

그 외에도 여러 이유가 있으며, 결국 우리는 아래와 같은 형태로 애플리케이션을 업데이트하려고 한다.

<img src="https://docs.docker.com/get-started/images/multi-app-architecture.png"/>

## 컨테이너 네트워킹

기본적으로, 컨테이너는 격리된 환경에서 실행되며, 동일한 머신 내의 다른 프로세스나 컨테이너에 대해서는 아무것도 알지 못한다는 점을 기억하라. 그렇다면, 어떻게 컨테이너 상호 간의 소통을 주도할 수 있을까? 정답은 **Networking**이다. **두 컨테이너가 같은 네트워크 상에 있다면, 컨테이너 간에 상호작용을 할 수 있다. 그렇지 않다면, 불가능하다.**

## MySQL 실행

컨테이너를 네트워크에 포함시키기 위한 두가지 방법이 있다. **1) 실행 시점에 할당시키거나**, **2) 기존 컨테이너에 연결한다.** 먼저, 네트워크를 생성하고 MySQL 컨테이너를 해당 네트워크에 첨부해보자.

1. 네트워크를 생성한다.

```bash
docker network create todo-app
```

2. MySQL 컨테이너를 실행하고 네트워크에 첨부한다. 데이터베이스를 초기화하기 위해서 환경 변수를 지정해주어야 한다.

```bash
 docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
```

PowerShell을 이용한다면 아래 명령을 사용하자.

```shell
 docker run -d `
     --network todo-app --network-alias mysql `
     -v todo-mysql-data:/var/lib/mysql `
     -e MYSQL_ROOT_PASSWORD=secret `
     -e MYSQL_DATABASE=todos `
     mysql:5.7
```

위에서 `--network-alias` 플래그를 지정한 것을 볼 수 있는데, 이에 대해서는 아래쪽에서 다루도록 하자.

> **유의** : 위에서 `todo-mysql-data`로 볼륨명을 지정하고, `/var/lib/mysql`에 마운트한 것을 확인할 수 있는데, 이는 MySQL이 데이터를 저장하는 디렉토리이다. 헌데, 이 후 `docker volume create` 명령을 수행하지는 않는다. Docker가 Named volume의 사용을 인지하고 자동으로 볼륨을 생성해주기 때문이다.

3. 이후 데이터베이스를 실행하고, 연결한 후에 제대로 연결되었는지를 확인해보자.

```bash
docker exec -it <mysql-container-id> mysql -u root -p
```

패스워드 프롬프트가 뜨면, 패스워드를 입력한다. 이후 MySQL 셸에서 데이터베이스를 리스트하고 `todos` 데이터베이스를 확인하자.

```bash
mysql> SHOW DATABASES;
```

이제 아래와 같은 결과가 보일 것이다.

```bash
 +--------------------+
 | Database           |
 +--------------------+
 | information_schema |
 | mysql              |
 | performance_schema |
 | sys                |
 | todos              |
 +--------------------+
 5 rows in set (0.00 sec)
```

여기까지가 `todos` 데이터베이스를 만드는 과정이었다.

## MySQL에 연결

이제 MySQL에 DB를 구성했고, 이제 사용하기만 하면 된다. 문제는 이를 어떻게 사용하느냐인데, 동일한 네트워크에서 다른 컨테이너들을 어떻게 찾아낼 수 있을까?

이를 확인하기 위해서 [nicolaka/netshoot](https://github.com/nicolaka/netshoot) 컨테이너를 사용한다. 해당 컨테이너는 네트워킹 이슈에 대한 트러블 슈팅 혹은 디버깅에 유용한 수많은 툴을 제공한다.

1. nicolaka/netshoot 이미지를 사용하여 새로운 컨테이너를 실행한다. 기존에 생성한 것과 동일한 네트워크에 연결하는 것임을 확인하자.

```bash
docker run -it --network todo-app nicolaka/netshoot
```

2. 컨테이너 내에서 `dig` 명령을 사용하는데, 이는 유용한 DNS 툴이다. 아래 명령으로 `mysql`이라는 호스트네임에 대한 IP 주소를 찾을 수 있다.

```bash
dig mysql
```

그리고 그 결과는 아래처럼 나타난다.

```bash
 ; <<>> DiG 9.14.1 <<>> mysql
 ;; global options: +cmd
 ;; Got answer:
 ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
 ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

 ;; QUESTION SECTION:
 ;mysql.				IN	A

 ;; ANSWER SECTION:
 mysql.			600	IN	A	172.23.0.2

 ;; Query time: 0 msec
 ;; SERVER: 127.0.0.11#53(127.0.0.11)
 ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
 ;; MSG SIZE  rcvd: 44
```

`ANSWER SECTION` 부분에서 `mysql`의 `A` 레코드가 `172.23.0.2`로 지정되어 있음을 확인할 수 있다. (환경에 따라 IP 주소는 달라질 수 있다.) `mysql`은 일반적으로 타당한 호스트네임이 아니지만, Docker는 network alias를 보유한 컨테이너의 IP 주소를 사용함으로써 이를 처리했다. (앞서 `--network-alias` 플래그를 사용했던 것을 기억하자.)

그 결과, 이제 애플리케이션은 `mysql`이라는 이름의 호스트에 연결하기만 하면, 데이터베이스와 상호작용할 수 있게 된다.

## MySQL과 함께 애플리케이션 구동

이제 TODO 앱은 MySQL과 연결하기 위해 몇가지 환경 변수 설정이 필요하다.

- `MYSQL_HOST`
- `MYSQL_USER`
- `MYSQL_PASSWORD`
- `MYSQL_DDB`

> **환경 변수에 대해** : 개발 단계에서 환경 변수를 사용하는 것은 괜찮지만, 애플리케이션이 프로덕션 단계에서 실행되는 경우 환경 변수는 **대부분의 경우 사용하지 말아야 한다.** 그 이유에 대해서는 Docker의 보안 리드 Diogo Monica의 [블로그 포스트](https://diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)를 참조하자.
>
> 보다 안전한 방법은, 컨테이너 오케스트레이션 프레임워크에서 제공하는 Secret Support 기능을 사용하는 것이다. 대부분의 경우 이러한 Secret들은 실행 중인 컨테이너에 파일로 마운트된다. 많은 애플리케이션들이 이를 위해 `_FILE` 접미사가 붙은 변수들을 지원하는 것을 확인할 수 있다.
>
> 예를 들어, 우리가 사용하는 예시 애플리케이션에서는 `MYSQL_PASSWORD_FILE` 변수를 설정하여 DB를 연결하기 위한 환경변수가 담긴 파일을 참조하도록 할 수 있다. Docker 자체는 별도로 환경 변수를 지원하지 않는다. 애플리케이션 자체적으로 사용할 환경 변수에 대한 파일을 찾아서 사용하도록 구현해야 한다.

당장에는 환경 변수를 통해서 애플리케이션을 DB에 연결해보자.

1. 각각의 환경 변수들을 작성해주고, 컨테이너를 네트워크에 연결해준다.

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

PowerShell을 사용한다면 아래와 같이 작성한다.

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

2. 컨테이너 로그를 살펴보면(`docker logs <container-id>`), 다음과 같이 DB 연결에 성공했음을 나타내는 메시지를 확인할 수 있다.

```bash
 # Previous log messages omitted
 $ nodemon src/index.js
 [nodemon] 1.19.2
 [nodemon] to restart at any time, enter `rs`
 [nodemon] watching dir(s): *.*
 [nodemon] starting `node src/index.js`
 Connected to mysql db at host mysql
 Listening on port 3000
```

3. 브라우저에서 애플리케이션을 실행하고, TODO앱을 테스트해본다.

4. 아래 명령을 통해 mysql 데이터베이스에 연결하고, 데이터가 적절하게 저장되었는지를 확인하자.

```bash
docker exec -it <mysql-container-id> mysql -p todos
```

mysql 셸 상에서 아래와 같이 작성하고, 결과를 확인한다.

```bash
 mysql> select * from todo_items;
 +--------------------------------------+--------------------+-----------+
 | id                                   | name               | completed |
 +--------------------------------------+--------------------+-----------+
 | c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
 | 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
 +--------------------------------------+--------------------+-----------+
```

현재 Docker 대쉬보드를 확인해본다면, 두 개의 앱 컨테이너가 작동하고 있음을 확인할 수 있다. 하지만, 두 컨테이너가 하나의 애플리케이션을 위해 그룹화되어 있음을 확인할 수는 없다. 이를 개선할 방법에에 대해서는 이후 챕터에서 알아보겠다.

<img src="https://docs.docker.com/get-started/images/dashboard-multi-container-app.png" />
