# Docker로 Redis 띄우기



## Redis란

Remote Dictionary Server의 약자로, 인메모리 기반 NoSQL 데이터베이스 시스템이다. 

(데이터가 디스크가 아닌 메모리에 저장됨 → 수용력은 적지만 접근 속도 빠름, 프로세스가 죽으면 데이터가 사라짐.)

(캐시, Message broker 용도로 많이 사용된다.)



## Docker로 Redis 띄우기

```bash
$ sudo docker pull redis   # image pull

$ mkdir redis && cd redis

$ sudo vi redis.conf   # redis.conf 파일을 로컬에 저장
# NETWORK
port 6005

# SECURITY
requirepass qwer1234!

# CLIENTS
maxclients 10000

$ sudo docker run --name my-redis -p 6379:6005 -v /home/ubuntu/redis/redis.conf:/usr/local/etc/redis/redis.conf redis redis-server   # 컨테이너 실행과 동시에 redis-server 명령 실행

$ sudo docker exec -it my-redis redis-cli # redis-cli로 접속

```

