# Apache - PHP 컨테이너 빌드하여 실행하기



#### 1. 디렉터리 만들기

```bash
cd ~
mkdir -p 3week/apache
cd 3week/apache
mkdir html
vi Dockerfile
```



#### 2. vi Dockerfile

ubuntu에 apache + php를 설치하는 이미지입니다.

```bash
FROM ubuntu:18.04

# Avoiding user interaction with tzdata
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y apache2 # install Apache web server (Only 'yes')
RUN apt-get install -y software-properties-common
RUN add-apt-repository ppa:ondrej/php # For Installing PHP 5.6
RUN apt-get update && RUN apt-get install -y php5.6

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
```



#### 3. Build

```bash
sudo docker build -t myapache-php .
```



#### 4. myapache-php 실행 (컨테이너)

```bash
sudo docker run -p 80:80 -v /home/ubuntu/3week/apache/html:/var/www/html --name mycontainer myapache-php
```



#### 5. AWS에 접속하여 인바운드 규칙 - 80번 포트를 열어주기

인스턴스 - 네트워크 및 보안 - 보안그룹 - 인바운드 규칙 편집



#### 6. 본인의 인스턴스 IP에 접속해보기



#### 7. Host OS의 index.php 수정

```bash
cd /home/ubuntu/3week/apache/html 
sudo vim index.php
```

```bash
<?php phpinfo(); ?>
```



#### 8. 본인의 인스턴스 IP에 다시 접속해보기

