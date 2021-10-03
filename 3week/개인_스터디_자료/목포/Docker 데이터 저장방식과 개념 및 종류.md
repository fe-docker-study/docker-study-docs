# Docker 데이터 저장방식과 개념 및 종류

Docker를 사용하다보면 데이터 저장 관련 문제와 맞닥뜨리는 경우가 있다. `컨테이너를 삭제하면 컨테이너에 쓰여진 데이터도 함께 삭제`되고, `다른 프로세스에서 특정 컨테이너를 사용하기 어려운` 경우가 생기기도 한다.
Docker에서는 이와 같은 문제를 위해 세 가지 옵션을 제공한다.

1. Docker **볼륨**을 이용한 방식
2. 호스트 서버의 경로와 컨테이너 경로를 연결하는 **바인드 마운트** 방식
3. tmpFS Mount

## Docker 볼륨을 이용한 방식

Docker의 볼륨은 기본적으로 /var/lib/docker/volumes 경로에 저장된다. 각 데이터는 /var/lib/docker/volumes/볼륨명/\_data 아래에 저장된다. 이 볼륨들은 컨테이너가 삭제되어도 유지된다.

![image](https://user-images.githubusercontent.com/31172248/135749401-24aed981-5467-4d55-80ca-cf4df4cb1949.png)

### Volume 조회

```console
$ docker volume ls

DRIVER    VOLUME NAME
local     04792ab55e833b72fececf3971c56fb636e0b0b626bd203ff617f204535a506d
local     c4354c641063eed1058d100861945b9e82a744f6ce780e64d34e356243635184
local     docker-hadoop_hadoop_datanode
local     docker-hadoop_hadoop_namenode
local     hadoop3_datanode01
local     hadoop3_datanode02
local     hadoop3_datanode02i
local     hadoop3_namenode

$ sudo docker volume inspect hadoop3_namenode

[
    {
        "CreatedAt": "2021-09-26T12:15:53Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "hadoop3",
            "com.docker.compose.version": "1.29.2",
            "com.docker.compose.volume": "namenode"
        },
        "Mountpoint": "/var/lib/docker/volumes/hadoop3_namenode/_data",
        "Name": "hadoop3_namenode",
        "Options": null,
        "Scope": "local"
    }
]

```

### 볼륨 생성하고 컨테이너에 마운트하기

만약 새 컨테이너에 기존 볼륨을 연결하고 싶을 때 마운트 기능을 사용한다. docker run 으로 컨테이너 실행 시 -v로 볼륨 경로를 명시하면 된다.

```console
$ docker create volume <볼륨 이름>

$ docker run -v <볼륨 이름>:/<마운트 될 경로> --name <연결할 컨테이너 이름>

// 사용 예
// new-v 라는 볼륨을 one 컨테이너에 마운트하고 app 경로에 test.txt 파일 추가
$ docker run -v new-v:/app --name one busybox touch /app/test.txt

// /app 경로에 test.txt를 생성했기 때문에 new-v 볼륨에도 추가된다.
$ ls /var/lib/docker/volumes/new-v/_data
test.txt
```

### 다른 컨테이너에도 마운트해보기

위에서 생성한 볼륨을 다른 컨테이너에도 마운트할 수 있다.

```console

// two 컨테이너에 new-v 볼륨을 연결하고 ls 명령어로 /app 경로를 확인한다.
$ docker run -v new-v:/app --name two busybox ls /app
test.txt
```

동일한 볼륨을 사용했기 때문에 기존에 생성한 test.txt 파일을 볼 수 있다.

## Bind Mount (호스트 서버 경로와 컨테이너의 경로를 연결)

![image](https://user-images.githubusercontent.com/31172248/135749415-52a784f9-f170-419a-9c13-d9124e823ac7.png)

-v 옵션의 콜론(:) 앞 부분에 마운트명 대신 호스트의 경로를 써주면된다.

### 컨테이너에 마운트

```
// 현재 경로(볼륨용으로 사용할 host 경로) 에 test.txt를 생성하고
// 컨테이너의 /app 경로와 연결해준 뒤 ls로 /app 경로 조회
$ touch test.txt
$ docker run -v `pwd`:/app -it --name one ls /app
test.txt
```

반대로 컨테이너 /app 경로에 test2.txt를 생성하고 host 경로를 조회하면 test2.txt가 존재하는 걸 확인할 수 있다.

### volume mount 경로 확인하기

3주차에 진행한 apahce 컨테이너로 예시를 들어보자

```
$ sudo docker run -p 80:80 -v /home/ubuntu/3week/apache/html:/var/www/html --name mycontainer myapache-php

// 다음과 같이 host 경로와 volume 마운트 경로를 지정해 실행한 뒤
$ docker inspect <container ID>
.
.
"Mounts": [
    {
        "Type": "bind",
        "Source": "/home/ubuntu/apache/html",
        "Destination": "/var/www/html",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
.
.
// 다음과 같이 host의 볼륨 경로와 conatiner의 마운트 경로를 확인할 수 있다.
```

## 볼륨 vs Bind Mount (What is difference?)

두 방식의 차이점은 Docker가 해당 마운트 포인트를 관리해주느냐 안 해주느냐이다. 볼륨을 사용할 시 직접 볼륨을 생성하거나 삭제해야 하지만, Docker 상에서 해당 볼륨을 관리해준다. (image나 container처럼) 그래서 대부분의 상황에서는 볼륨을 사용하는 것이 권장되지만 컨테이너화된 로컬 개발 환경을 구성할 때는 바인드 마운트가 더 유리할 수 있다. (변경사항을 바로바로 확인할 수 있기 때문에)

## mount 옵션

우리는 위에서 볼륨 생성 시 -v 옵션을 사용한다고 배웠다. 이는 독립형 컨테이너에서 사용되는 방식이며 **Docker Swarm Mode**(호스트 서버의 컨테이너들을 배포, 관리하기 위한 툴. 쿠버네티스를 대체할 수 있는 툴이라고 보면 됨)에서는 --mount가 사용된다.
--mount 옵션은 쉼표로 구분되며 여러 key-value 쌍으로 구성된다.

## tmpFS Mount

![image](https://user-images.githubusercontent.com/31172248/135749421-5165e2bd-0045-4d9c-a41e-78d84f23118e.png)

tmpfs mount는 호스트 시스템이나 컨테이너 내에서 데이터를 유지하지 않으려는 경우에 적합하다. 이는 보안상의 이유이거나 애플리케이션이 많은 양의 비영구상태 데이터를 작성해야할 때 컨테이너의 성능을 보호하기 위한 것일 수도 있다.

하지만, 이 방법은 볼륨, Bind Mount와 같이 다른 컨테이너에 tmpfs 마운트를 공유할 수 없다. 또한, Linux에서 Docker를 실행하는 경우에만 사용할 수 있다.

### 출처

https://www.daleseo.com/docker-volumes-bind-mounts/

https://watch-n-learn.tistory.com/34

https://hello-i-t.tistory.com/m/130
