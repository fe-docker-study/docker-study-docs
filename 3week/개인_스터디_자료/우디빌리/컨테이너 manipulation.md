# 컨테이너

### 컨테이너 실행

docker <object-type> <command> <options>

- `object-type` :  조작할 Docker Object.  예) container, image, metwork, volume
- `command` : daemon이 수행할 작업. 예) run
- `options` :  예) `—publish`  option for port mapping

docker container run <image name>

### publishing port

- 컨테이너는 독립된 환경임. 따라서 host system은 container 안에서 무슨 일이 일어나는지 전혀 모른다. 따라서, 컨테이너 내부의 application은 외부에서 접근할 수 없는 상태이다.
- 외부에서 컨테이너 내부의 application에 접근할 수 있도록 하기 위해, 반드시 컨테이너 내부의 포트를 host의 local network에 publish 해야한다.

-p <host port>:<container port>

### 컨테이너 중지

docker container stop(or kill) <container identifier>

### 컨테이너 재시작

docker container start <container identifier>
