# Docker File로 Hadoop 환경 구축하기

## Docker 재설치

(https://docs.docker.com/engine/install/ubuntu/)

## Test Environment

- Hadoop 3.1.1
- Docker version 20.10.8
- docker-compose 1.29.2

## Dockerfile

- 베이스가 될 Docker Image
- Docker 컨테이너 안에서 수행할 조작(명령)
- 환경변수 등의 설정
- Docker 컨테이너 안에서 작동시켜둘 데몬 실행

### 기본 구문

| 제목        | 내용                       |
| :---------- | :------------------------- |
| FROM        | 베이스 이미지 지정         |
| RUN         | 명령 실행                  |
| CMD         | 컨테이너 실행 명령         |
| LABEL       | 라벨 설정                  |
| EXPOSE      | 포트 익스포트              |
| ENV         | 환경변수                   |
| ADD         | 파일/디렉토리 추가         |
| VOLUME      | 볼륨 마운트                |
| USER        | 사용자 지정                |
| WORKDIR     | 작업 디렉토리              |
| ARG         | Dockerfile 안의 변수       |
| ONBUILD     | 빌드 완료 후 실행되는 명령 |
| STOPSIGNAL  | 시스템 콜 시그널 설정      |
| HEALTHCHECK | 컨테이너의 헬스체크        |
| SHELL       | 기본 쉘 설정               |

<br/>

## 파일 구조

📦hadoop3  
 ┣ 📂base  
 ┃ ┣ 📜Dockerfile  
 ┃ ┗ 📜core-site.xml  
 ┣ 📂datanode  
 ┃ ┣ 📜Dockerfile  
 ┃ ┣ 📜hdfs-site.xml  
 ┃ ┗ 📜start.sh  
 ┗ 📂namenode  
 ┃ ┣ 📜Dockerfile  
 ┃ ┣ 📜hdfs-site.xml  
 ┃ ┗ 📜start.sh

## 파일 작성

### base image 생성

#### /hadoop3/base/Dockerfile

```Dockerfile
# 기반이 되는 이미지를 설정. ubuntu 18 기반인 zulu의 openjdk 8버전을 선택
FROM azul/zulu-openjdk:8
RUN apt-get update && apt-get install -y curl openssh-server wget vim

ENV HADOOP_VERSION=3.1.1
ENV HADOOP_URL=https://archive.apache.org/dist/hadoop/core/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz

# SSH 관련 설정
RUN mkdir /var/run/sshd

# Hadoop 내려받고 /opt/hadoop에 압축 해제
RUN curl -fSL "$HADOOP_URL" -o /tmp/hadoop.tar.gz \
    && tar -xvf /tmp/hadoop.tar.gz -C /opt/ \
    && rm /tmp/hadoop.tar.gz

# 데이터 디렉토리 생성 및 설정 폴더의 심볼릭 링크 생성
RUN ln -s /opt/hadoop-$HADOOP_VERSION /opt/hadoop \
    && mkdir /opt/hadoop/dfs \
    && ln -s /opt/hadoop-$HADOOP_VERSION/etc/hadoop /etc/hadoop \
    && rm -rf /opt/hadoop/share/doc \

# 로컬의 core-site.xml 파일을 복제
ADD core-site.xml /etc/hadoop/

# 실행 환경에 필요한 환경 변수 등록
ENV HADOOP_PREFIX /opt/hadoop
ENV HADOOP_CONF_DIR /etc/hadoop
ENV PATH $HADOOP_PREFIX/bin/:$PATH
ENV JAVA_HOME=/usr/lib/jvm/zulu8-ca-amd64
EXPOSE 22
CMD /usr/sbin/sshd -D
```

<br/>

#### /hadoop3/base/core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://host:9000/</value>
    <description>NameNode URI
    </description>
  </property>
</configuration>
```

#### 이미지 빌드

```
cd base
sudo docker build -t hadoop-base:3.1.1 .
```

#### 이미지 확인

```
sudo docker images
```

<br/>

<img src="https://images.velog.io/images/shinmj1207/post/e11dd3f1-b80a-4d90-b91a-14a7d8e4fd02/image.png" style="width : 30%;">

### hadoop namenode image 생성

#### /hadoop3/namenode/Dockerfile

```Dockerfile
# 로컬에 생성한 base 이미지
FROM hadoop-base:3.1.1

# NameNode Web UI 응답 여부를 통해 Healthcheck
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 CMD curl -f http://localhost:9870/ || exit 1

# 설정 파일 복제
ADD hdfs-site.xml /etc/hadoop/

# FsImage, EditLog 파일 경로를 volume으로 연결
RUN mkdir /opt/hadoop/dfs/name
VOLUME /opt/hadoop/dfs/name

# 실행 스크립트 복제
ADD start.sh /start.sh
RUN chmod a+x /start.sh

# NameNode의 HTTP, IPC 포트 노출
EXPOSE 9870 9000

# 시작 명령어 등록
CMD ["/start.sh", "/opt/hadoop/dfs/name", "/usr/sbin/sshd"]
```

#### /hadoop3/namenode/hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///opt/hadoop/dfs/name</value>
  </property>
  <property>
    <name>dfs.blocksize</name>
    <value>10485760</value>
  </property>
  <property>
    <name>dfs.client.use.datanode.hostname</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-bind-host</name>
    <value>0.0.0.0</value>
  </property>
  <property>
    <name>dfs.namenode.servicerpc-bind-host</name>
    <value>0.0.0.0</value>
  </property>
  <property>
    <name>dfs.namenode.http-bind-host</name>
    <value>0.0.0.0</value>
  </property>
  <property>
    <name>dfs.namenode.https-bind-host</name>
    <value>0.0.0.0</value>
  </property>
</configuration>
```

#### /hadoop3/namenode/start.sh

```shell
#!/bin/bash

NAME_DIR=$1
echo $NAME_DIR

# 비어있지 않다면 이미 포맷된 것이므로 건너뛰고
if [ "$(ls -A $NAME_DIR)" ]; then
  echo "NameNode is already formatted."

# 비어있다면 포맷을 진행
else
  echo "Format NameNode."
  $HADOOP_PREFIX/bin/hdfs --config $HADOOP_CONF_DIR namenode -format
fi

# NameNode 기동
$HADOOP_PREFIX/bin/hdfs --config $HADOOP_CONF_DIR namenode
```

#### namenode 이미지 빌드

```
cd namenode
sudo docker build -t hadoop-namenode:3.1.1 .
```

<br/>

<img src = "https://images.velog.io/images/shinmj1207/post/60bc83c0-b600-4b6a-a7d1-42cbf3e64b1e/image.png" style="width : 30%;">

<br/><br/>

### hadoop datanode image 생성

#### /hadoop3/datanode/Dockerfile

```Dockerfile
FROM hadoop-base:3.1.1

# DataNode Web UI 응답 여부를 통해 Healthcheck
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3  CMD curl -f http://localhost:9864/ || exit 1
RUN mkdir /opt/hadoop/dfs/data
VOLUME /opt/hadoop/dfs/data
ADD start.sh /start.sh
RUN chmod a+x /start.sh

# WebUI, 데이터전송
EXPOSE 9864 9866
CMD ["/start.sh", "/usr/sbin/sshd"]
```

#### /hadoop3/datanode/hdfs-site.xml

```xml
<configuration>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///opt/hadoop/dfs/data</value>
  </property>
  <property>
    <name>dfs.blocksize</name>
    <value>10485760</value>
  </property>
  <property>
    <name>dfs.datanode.use.datanode.hostname</name>
    <value>true</value>
  </property>
</configuration>
```

#### /hadoop3/datanode/start.sh

```shell
#!/bin/sh
$HADOOP_PREFIX/bin/hdfs --config $HADOOP_CONF_DIR datanode
```

#### datanode 이미지 빌드

```
cd datanode
sudo docker build -t hadoop-datanode:3.1.1 .
```

<br/>

### docker-compose.xml 작성

### Docker Compose?

Docker Compose는 여러 컨테이너를 모아서 관리하기 위한 툴이다. docker-compose.yml이라는 파일에 컨테이너 구성 정보를 정의함으로써 동일 호스트상의 여러 컨테이너를 일괄적으로 관리한다. 파일에는 관리하고 싶은 컨테이너의 서비스, 네트워크, 볼륨 등을 정의하면 된다.

![](https://images.velog.io/images/shinmj1207/post/b5efd30c-8214-490c-9201-d0199e68f7a4/image.png)

#### DockerCompose 설치

(https://docs.docker.com/compose/install/)

```
 $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

 $ sudo chmod +x /usr/local/bin/docker-compose

 $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

 $ docker-compose --version
```

#### /hadoop3/docker-compose.yml

```yml
# Docker Compose의 버전 (https://docs.docker.com/compose/compose-file/)
version: "3.8"

x-datanode_base: &datanode_base
  image: hadoop-datanode:3.1.1
  networks:
    - bridge

#각 컨테이너에 사용할 이미지와 포트, 볼륨 정보를 정의
services:
  namenode:
    image: hadoop-namenode:3.1.1
    container_name: namenode
    hostname: public host
    ports:
      - 9870:9870
      - 9000:9000
    volumes:
      - /data/hadoop_data/name:/opt/hadoop/dfs/name
      - /tmp:/tmp
    networks:
      - bridge
  datanode01:
    <<: *datanode_base
    container_name: datanode01
    hostname: public host
    volumes:
      - /data/hadoop_data/name:/opt/hadoop/dfs/data
  datanode02:
    <<: *datanode_base
    container_name: datanode02
    hostname: public host
    volumes:
      - /data/hadoop_data/name:/opt/hadoop/dfs/data
volumes:
  namenode:
  datanode01:
  datanode02:
networks:
  bridge:
```

<br/>

#### Docekr Network (https://bluese05.tistory.com/15)

![](https://t1.daumcdn.net/cfile/tistory/2750F23D55A37EF801)

#### docker-compose 실행

```
sudo docker-compose -f docker-compose.yml up -d
```

## 컨테이너 확인

```
sudo docker ps
```
