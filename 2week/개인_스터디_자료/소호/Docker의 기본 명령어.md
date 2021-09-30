`이 글은 생활코딩(egoing)의 Docker 입문수업을 참고하여 정리한 자료입니다. (하단 참고자료 표기)`

----
# Docker의 기본 명령어



(Docker 설치는 알아서 했다고 치고) Docker의 기본 명령어를 알아보자...👍
설치 매뉴얼 : <https://docs.docker.com/engine/install/ubuntu/>



## 1) docker pull
Docker hub 로부터 이미지를 받는 명령어.
```bash
# docker pull <이미지명>[:태그명]

# 이미지명 앞에 사용자명을 지정하면 Docker hub에 해당 사용자가 올린 이미지를 받을 수 있음
# 공식 이미지에는 사용자명이 붙지 않음
$ docker pull monadk/myubuntu
$ docker pull httpd  # 아파치 official image 다운로드

# 태그에 latest를 설정하면 최신버전을 받음
$ docker pull ubuntu:latest
```


## 2) docker images

받은 이미지의 목록을 출력하는 명령어.
```bash
$ docker images
```


## 3) docker run

이미지를 컨테이너로 생성하여 실행하는 명령어.
run하면 실행된 컨테이너의 로그가 출력된다.

```bash
# docker run [options] <이미지명> [command]
$ docker run httpd

# 컨테이너의 이름 지정 옵션 : --name <컨테이너명>
$ docker run --name monad_apache httpd  # monad_apache라는 컨테이너명으로 생성
```


## 4) docker ps

현재 실행중인 컨테이너 목록을 출력하는 명령어.
```bash
$ docker ps

# 모든 상태의 컨테이너 목록을 출력하는 옵션 : -a
$ docker ps -a
```


## 5) docker stop

실행중인 컨테이너를 중지하는 명령어.
중지된 컨테이너는 위의 "docker ps -a"를 통해 확인할 수 있다.

```bash
# docker stop <컨테이너명 or 컨테이너ID>
$ docker stop monad_apache
```


## 6) docker start

중지된 컨테이너를 시작하는 명령어.
이 때는 컨테이너의 로그 출력이 안 됨.

```bash
# docker start <컨테이너명 or 컨테이너ID>
$ docker start monad_apache
```


## 7) docker logs

컨테이너의 로그를 출력하는 명령어.
```bash
# docker logs <컨테이너명 or 컨테이너ID>
$ docker logs monad_apache   # 한 번 출력되고 끝나버림..

# forever 옵션 : -f
$ docker logs -f monad_apache   # 새로 작성되는 로그가 계속 출력됨
```


## 8) docker rm

컨테이너를 삭제하는 명령어.
```bash
# docker rm <컨테이너명 or 컨테이너ID>
$ docker rm monad_apache  # 실행중인 컨테이너는 삭제가 안됨 (중지된 컨테이너만)

# force 옵션 : --force (실행중인 컨테이너도 강제로 삭제)
$ docker rm --force monad_apache
```


## 9) docker rmi

이미지를 삭제하는 명령어.
```bash
# docker rmi <이미지명>
$ docker rmi httpd
```






---

`※참고 자료※`
<https://www.youtube.com/watch?v=iLcUr0EQdrM&list=PLuHgQVnccGMDeMJsGq2O-55Ymtx0IdKWf&index=4>