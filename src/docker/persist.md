## 컨테이너 파일시스템

컨테이너가 실행될 때, 파일시스템을 구축하기 위해 이미지로부터 여러 개의 레이어를 사용한다. 각각의 컨테이너는 파일을 생성/업데이트/제거하기 위한 "Scratch space"를 갖는다. 한 컨테이너 내의 어떤 변화는 다른 컨테이너에 영향을 주지 않으며, 심지어 그것이 같은 이미지로부터 만들어진 컨테이너라도 마찬가지다.

## 실전

직접 두 개의 컨테이너를 실행시키고 각각 하나의 파일을 만들게끔 해보자. 이로부터 하나의 컨테이너에서 생긴 파일은 다른 컨테이너에서 활용할 수 없음을 확인할 수 있을 것이다.

1. `ubuntu` 컨테이너를 실행하고 1에서 10000 사이의 난수를 갖는 `/data.txt`라는 이름의 파일을 만든다.

```bash
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

위에서는 `&&`를 통해 두 가지 명령을 실행했는데, 첫번째는 무작위 번호를 추출하여 `/data.txt`라는 파일로 작성한 것이고, 두번째는 컨테이너를 실행 상태로 유지하기 위해 파일을 확인하는 것이다.

2. 컨테이너에 `exec`을 해줌으로써 결과를 확인할 수 있다. 그렇게 하기 위해서, 대쉬보드를 열고 실행 중인 `ubuntu` 이미지가 실행 중인 컨테이너를 클릭하자.

<img src="https://docs.docker.com/get-started/images/dashboard-open-cli-ubuntu.png"/>

그러면 현재 실행 중인 `ubuntu` 컨테이너 내에서 동작하고 있는 터미널을 확인할 수 있다. 작성한 `/data.txt` 파일을 확인하기 위해 아래의 명령을 실행하고, 터미널을 다시 종료하자.

```bash
cat /data.txt
```

만약, 대쉬보드보다 CLI 방식을 더 선호한다면, 아래와 같이 `docker exec` 명령을 실행하여 똑같이 진행할 수 있다. 이 경우 컨테이너의 ID를 알고 있어야 한다.(`docker ps`를 사용하자.)

```bash
docker exec <container-id> cat /data.txt
```

그러면 `/data.txt`에 작성된 난수를 확인할 수 있을 것이다!

3. 이제, 똑같은 이미지를 통해 다른 `ubuntu` 컨테이너를 실행시켜보자. 동일한 파일이 존재하지 않음을 알 수 있다.

```bash
docker run -it ubuntu ls /
```

보시다시피 `data.txt`가 존재하지 않는다. 말했다시피 해당 파일은 첫번째 컨테이너에 대한 "scratch space"에 작성되었기 때문이다.

## 컨테이너 볼륨

앞선 실험에서, 동일한 이미지에서 실행한 각각의 컨테이너는 매번 새롭게 실행되는 것임을 확인했다. 각각의 컨테이너 내에서 일어나는 일련의 CRUD 작업들은 해당 컨테이너 내에서만 영향을 준다. 단, **볼륨(Volume)**을 사용한다면, 이를 바꿀 수 있다.

[볼륨](https://docs.docker.com/storage/volumes/)은 컨테이너의 특정 파일 시스템 경로를 호스트 머신에 연결시켜줄 수 있게 해준다. 컨테이너의 디렉토리가 마운트되면, 디렉토리 내의 변경사항들은 호스트 머신에도 적용된다. 덕분에, 컨테이너가 여러번 재실행되더라도, 동일한 디렉토리를 마운트하게 되면 매번 동일한 파일을 유지할 수 있다.

볼륨에는 두 가지 종류가 있는데, 하나는 **Named volumes**이다.

## 데이터 유지하기

앞선 챕터에서, 기본적으로 우리의 TODO 앱은 [SQLite](https://www.sqlite.org/index.html)를 통해 `/etc/todos/todo.db`에 데이터를 저장한다. SQLite는 하나의 파일에 데이터를 저장하는 간단한 형태의 관계형 DB다. 이는 대규모의 애플리케이션에 적합하진 않지만, 작은 데모에서는 잘 동작한다. DB 엔진을 변경하는 방법에 대해서는 추후에 따로 다루어보자.

DB가 하나의 파일이기 때문에, 해당 파일을 유지하기만 하면 다음 컨테이너의 실행에서도 DB에 저장된 내용을 유지할 수 있다. 볼륨을 만들고, 디렉토리에 첨부(일반적으로, **mouting**이라고 함)하면, 데이터가 저장, 유지된다. 현재 컨테이너는 `todo.db` 파일을 작성하기 때문에, 볼륨을 통해 해당 파일이 호스트에 지속될 것이다.

앞서 말했듯, 먼저 **named volume**을 사용해보겠다. named volume은 간단한 데이터 버킷이다. Docker가 디스크의 물리적인 로케이션을 유지하고, 우리는 해당 볼륨의 이름을 기억하기만 하면 된다. 해당 볼륨을 사용할 때마다, Docker가 적절한 데이터가 제공됨을 보장해줄 것이다.

1. `docker volume create` 명령으로 볼륨을 만든다.

```bash
docker volume create todo-db
```

2. 작동 중인 TODO 앱 컨테이너를 멈추고, 삭제한다. (대쉬보드, 혹은 `docker rm -f <id>`)

3. 새로 컨테이너를 실행하되, `-v` 플래그를 통해 마운트할 볼륨을 지정해준다. 여기선 named volume을 사용하고, 이를 `/etc/todos`에 마운트 해주었다. 이를 통해 해당 경로에 있는 모든 파일들이 캡처된다.

```bash
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

4. 애플리케이션을 실행하여 적절히 데이터가 생성 / 변경 / 유지되는지 확인한다.

<img src="https://docs.docker.com/get-started/images/items-added.png"/>

5. 2 ~ 4번을 다시 수행하면서, 컨테이너를 재실행시키더라도 데이터가 여전히 유지되는지 확인한다.

> **참고** : Docker에서는 기본적으로 named volume과 bind mounts(추후 설명)를 볼륨으로 제공하지만, NFS, SFTP, NetApp 등 수많은 볼륨 드라이버 플러그인들이 존재한다. 이는 Swarm, Kubernetes 등의 클러스터 환경을 통해 여러 호스트에서 컨테이너를 실행한다면 특히 중요하다.

## 볼륨 파헤치기

종종, "_Named volume을 사용할 때, Docker는 실제로 어디에 데이터를 저장하는 걸까?_"하는 물음이 들 수 있다. 이를 확인하고 싶다면, `docker volume inspect` 명령을 사용해보자.

```bash
docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

`Mountpoint`가 바로 데이터가 저장되는 디스크의 실제 위치다. 대부분의 머신에서는 해당 디렉토리에 접근하기 위해 루트 엑세스 권한이 요구된다.

> **Docker Desktop의 경우** : Docker Desktop을 실행하는 동안, Docker 명령은 실제로는 호스트 머신 내의 작은 VM 내에서 실행된다. 이 경우 `Mountpoint` 디렉토리 내의 실제 파일들을 확인하려고 한다면, 먼저 VM 내부로 들어가야 한다.
