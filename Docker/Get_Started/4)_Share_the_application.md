## 저장소(Repository) 생성

이미지를 푸쉬하기 위해서는 먼저 DockerHub에 저장소를 생성해야 한다.

1. Docker Hub에 가입하고, 로그인
2. **Create Repository** 버튼을 클릭
3. 저장소 이름을 설정하고, Visibility를 `Public`으로 지정
   > 기본적으로 Private 저장소는 개인 당 1개만 주어진다. 추가로 이용하거나 팀 별로 이용하고자 하는 경우엔 [Pricing](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade)을 참조하자.
4. **Create** 버튼을 클릭

해당 과정을 마쳤으면, 아래와 같이 **Docker commands**가 나타난다. 이는 현재 저장소에 푸쉬하기 위한 예시 명령이다.

<img src="https://docs.docker.com/get-started/images/push-command.png" />

## 이미지 푸쉬

1. 다음과 같이 푸쉬 명령을 작성한다. `docker`가 아니라, 본인의 네임 스페이스로 작성해주어야 함을 주의하자.

```bash
 $ docker push docker/getting-started
 The push refers to repository [docker.io/docker/getting-started]
 An image does not exist locally with the tag: docker/getting-started
```

뭐가 문제일까? 푸쉬 명령이 `docker/getting-started`라는 이름의 이미지를 찾아봤지만, 알 수 없었다. `docker image ls`를 실행해보면 알겠지만, 아무 것도 존재하지 않는다.

2. 먼저, `docker login -u <USER-NAME>` 명령으로 Docker Hub에 로그인한다.

3. `docker tag` 명령으로 `getting-started` 이미지에 새로운 이름을 부여한다.

```bash
docker tag getting-started <USER-NAME>/getting-started
```

4. 이제, 앞선 과정을 다시 해보자. 현재 따로 태그네임을 추가하지 않았으므로 `:tagname` 부분은 없어도 된다. 별도로 태그를 지정하지 않는 경우, Docker는 `latest`라는 이름의 태그를 사용한다.

```bash
docker push YOUR-USER-NAME/getting-started
```

## 새 인스턴스에 이미지 실행

이제, 우리가 빌드한 컨테이너 이미지를 전혀 다른 새로운 환경에서 사용해보자. 여기서는 **Play with Docker**를 사용한다.

1. 브라우저로 [Play with Docker](https://labs.play-with-docker.com/)에 접속한다.
2. **Login**을 클릭하고, **docker**를 선택한다.

3. 본인의 Docker Hub 계정으로 접속한다.

4. 로그인 한 후, **ADD NEW INSTANCE** 옵션을 클릭한다. 이 후, 브라우저 상에서 터미널을 확인할 수 있다.

<img src="https://docs.docker.com/get-started/images/pwd-add-new-instance.png" />

5. 해당 터미널에서 우리가 푸쉬했던 애플리케이션을 실행하자.

```bash
docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
```

6. 위쪽에 `3000` 포트를 클릭하면, 실행한 애플리케이션을 확인할 수 있다.
