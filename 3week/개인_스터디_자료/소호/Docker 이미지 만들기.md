# Docker 이미지 만들기



`이 글은 생활코딩(egoing)의 Docker 수업과 블로그들을 참고하여 정리한 자료입니다. (하단 참고자료 표기)`

----


이미지를 만드는 방법을 알아보자.👏

1. **Commit**을 통해 만드는 방법
2. **Build**를 통해 만드는 방법이 있다.



# 1. Commit을 통해 이미지 만들기
내가 ubuntu 이미지로 컨테이너를 만들어 git, python 등을 설치했을 때, 
이 실행환경을 그대로 백업하고 싶다면? 혹은 이미지로 만들어 배포하고 싶다면?
Commit을 하면 된다.

위에서 만든 container를 **Commit**하면 ubuntu+git+python이 설치된 이미지가 생성되는 것이다.
이 이미지를 docker hub과 같은 레지스트리에 배포하고 싶다면 해당 이미지를 **Push**하면 된다.

<img src="https://images.velog.io/images/monadk/post/199c2d4d-0dd3-49e1-a4e4-bb3371e70552/image.png" width="65%">






>## <실습해보기>
1. ubuntu 이미지를 실행
2. 실행된 컨테이너에 git 설치
3. ubuntu + git이 설치된 컨테이너를 commit하여 이미지 생성
4. 3의 이미지로 2개의 컨테이너를 실행하여 각각 python, node.js를 설치

<img src="https://images.velog.io/images/monadk/post/4f527e0c-7069-4b2d-8b9d-95ee28b374cb/image.png" width="65%">



### 1) ubuntu 이미지 실행

```bash
# ubuntu 이미지 다운로드
$ docker pull ubuntu

# my-ubuntu라는 컨테이너명으로 실행 (실행하자마자 bash 터미널에 연결 유지하기)
$ docker run -it --name my-ubuntu ubuntu bash
```


### 2) 실행된 컨테이너에 git 설치

```bash
# apt 갱신
$ apt update

# git 설치
$ apt install git
```


### 3) ubuntu+git이 설치된 컨테이너를 commit하여 이미지 생성

```bash
# 컨테이너 bash 연결 종료
$ exit

# commit!
# docker commit <컨테이너명> <리포지토리>:<태그>
$ docker commit my-ubuntu monadk:ubuntu-git

# 생성되었는지 확인
# monadk 리포지토리, ubuntu-git이라는 태그로 이미지 생성되어 있음
$ docker images
```


### 4) 2개의 컨테이너로 실행하여 각각 python, node.js를 설치

nodejs
```bash
# monadk:ubuntu-git 이미지를 nodejs라는 컨테이너명으로 실행
$ docker run -it --name nodejs monadk:ubuntu-git bash

# bash터미널 - nodejs 설치
$ apt update && apt install nodejs

# 실행 테스트 - 각각이 독립된 실행환경
$ nodejs  # 실행됨
$ python  # command not found
```
python
```bash
# monadk:ubuntu-git 이미지를 python 컨테이너명으로 실행
$ docker run -it --name python monadk:ubuntu-git bash

# bash터미널 - python 설치
$ apt update && apt install python

# 실행 테스트 - 각각이 독립된 실행환경
$ python  # 실행됨
$ nodejs  # command not found
```



# 2. Dockerfile & Build를 통해 이미지 만들기

위의 commit을 통해 이미지를 만드는 것은, 사용하고 있는 컨테이너를 **백업**한다는 뉘앙스에 가깝다.
**Dockerfile**을 이용해 Build 하는 것은 처음부터 **생성**을 하는 뉘앙스.
Dockerfile에 시간 순서에 따라 instructions가 명시되어 있기 때문에 어떤 이미지인지 가시적으로 알 수 있다.
`"Dockerfile"`은 약속된 파일명이다.

```bash
# Dockerfile의 기본적인 포맷

FROM ubuntu  # ubuntu 이미지를 기반으로 해서
RUN apt update && apt install -y git  # apt update하고, git을 설치해라
```


위의 Dockerfile을 빌드하는 명령어는 아래와 같다.
Dockerfile을 기반으로 이미지가 생성됨.

```bash
# docker build -t <리포지토리>:<태그> <Dockerfile이 위치하는 디렉터리>
# -t는 이미지이름 지정하는 옵션 (--tag)

$ docker build -t monadk:ubuntu-git-2 .
```



## Dockerfile 명령어 키워드

Dockerfile에서 사용되는 명령어 키워드를 간단하게 알아보자. 👍
- **FROM** : Dockerfile은 `FROM`으로 시작한다.
`FROM <IMAGE_NAME>`
커스텀 이미지의 기반이 되는 이미지 이름을 지정해준다. 
이미지를 생성할 때 FROM에 설정한 이미지가 로컬에 있으면 바로 사용하고, 없으면 Docker Hub에서 받아온다.

- **RUN** : 명령어를 실행하는 키워드. 쉘 스크립트 구문이 실행됨.
FROM으로 설정한 이미지에 포함된 /bin/sh 실행 파일을 사용하게 되며 /bin/sh 실행 파일이 없으면 사용할 수 없음.
`RUN <COMMAND>`
\** `RUN`이 실행될 때마다 Layer가 생성되어 저장된다.
- **WORKDIR** : 해당 디렉터리가 없다면 만들고, 사용자를 해당 디렉터리로 이동시킨다.
작업할 디렉터리를 설정하는 것이다.
`WORKDIR <PATH>`
- **COPY** : 호스트OS의 로컬 파일 또는 디렉토리를, 이미지(컨테이너) 내의 경로로 복사한다.
`COPY <Host의 복사할 파일 경로> <이미지에서 파일이 위치할 경로>`
- **CMD** : 컨테이너가 시작될 때 스크립트 혹은 명령을 실행한다. 
즉 `docker run` 명령으로 컨테이너를 생성하거나, `docker start` 명령으로 정지된 컨테이너를 시작할 때 실행되는 명령어를 지정. *`CMD`는 Dockerfile에서 한 번만 사용할 수 있다.*
`CMD <COMMAND>`

 \** `CMD`에 쓴 명령어는 컨테이너 실행(run)시에 오버라이딩 될 수 있다 ** 
 즉 `docker run --name web-server web-server-build pwd` 라는 명령어로 컨테이너가 실행될 때 pwd가 실행되도록 한다면, `CMD`에 설정된 커맨드 대신 pwd가 실행되는 것이다.

 \** `RUN`과 `CMD`의 차이점 **
 `RUN`은 Build가 되는 시점에 실행되는 명령어 -> 이미지에 반영된다.
 `CMD`는 Container가 실행되는 시점에 실행되는 명령어 -> 컨테이너에 반영된다.

 




---
`※참고 자료※`
<https://www.youtube.com/watch?v=RMNOQXs-f68>
<http://longbe00.blogspot.com/2015/03/dockerfile.html>

이고잉 님이 아래 블로그를 추천했는데 배운 내용 정리에 도움이 되었다. 👍
<https://www.44bits.io/ko/post/building-docker-image-basic-commit-diff-and-dockerfile#%EB%93%A4%EC%96%B4%EA%B0%80%EB%A9%B0>