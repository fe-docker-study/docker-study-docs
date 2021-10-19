# Docker Network 


//그림

## Docker Network의 구조
Docker 서비스를 설치하게 되면 자동으로 **Host Machine**의 Network Interface에 **Docker0** 라는 Virtural Interface가 생성된다. 생성 시 Gateway는 자동으로 172.17.0.1로 설정되며, 16 bit netmask(255.255.0.0)으로 설정된다. 
이렇게 자동으로 할당된 IP는 DHCP를 통해 받는 것이 아닌 docker 내부 로직에 의해 자동으로 할당된다.

Docker0는 각 Container의 virtual ethernet(veth) 인터페이스와 바인딩되며, 호스트 OS의 **eth0** 인터페이스와 이어주는 역할을 한다. (일종의 게이트웨이 역할) 이는 docker0가 **L2** 기반의 통신을 한다는 것을 알 수 있다.
Docker0는 bridge network를 지원해주기 위해 내부적으로 network address translation(NAT), 포트포워딩 기능을 지원한다. (물론 iptables를 통해서)

모든 컨테이너는 실행 시 docker0의 대역대 안에서 통신하며 virtual ethernet bridge는 172.17.0.0/16을 기본 대역으로 사용한다. 따라서, Container가 running을 하게되면 **172.17.X.Y** 형태로 IP 주소를 순차적 할당한다.


### Docker0의 구조
다음과 같은 명령어로 확인하면 docker 서비스 구동 시 docker0는 172.17.0.1/16의 대역을 가지고 있는 것을 확인할 수 있고, eth0는 host의 ip 대역을 가지고 있다는 것을 확인할 수 있다.
```docker
$ ipaddr

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:4c:57:1e brd ff:ff:ff:ff:ff:ff
    inet 14.49.44.164/23 brd 14.49.45.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe4c:571e/64 scope link
       valid_lft forever preferred_lft forever
.
.
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:85:44:51:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

## 컨테이너 포트를 외부로 노출 할 수 있나?
예를들어, 웹 기반의 Container가 존재한다고 가정하고 이를 외부에서 접속한다고 해보자. 우리는 컨테이너에 직접 접속할 수 있을까? 그럴 순 없다. 우리는 eth0(host)와 container간의 포트포워딩을 통해 이를 해결할 수 있다.


//그림

다음과 같이 nginx 컨테이너와 웹서버 appjs 가 하나씩 있다고 했을 때 컨테이너 실행 시 p옵션을 통해 **host포트:컨테이너 포트**를 매핑시켜줄 수 있다. 
```docker
$ docekr run --name web -d -p 80:80 nginx:1.14
$ iptables -t nat -L -n -v // iptable 방화벽롤이 만들어진 것을 확인할 수 있다.
```
* 여러 컨테이너가 같은 포트로 실행되는 것은 문제가 되지 않는다.(ex. nginx01(80), nginx02(80), nginx03(80)..) 이유는 컨테이너가 생성될 때마다 다른 IP가 할당되기 때문이다.
* 또한, 컨테이너의 포트만 설정해주면(-p 8080) host의 random port와 컨테이너 포트의 포트가 매핑된다.
* -P (대문자) 옵션만 설정 시 Dockerfile내의 EXPOSE로 정의된 포트가 host의 random포트와 매핑된다.

## 컨테이너 네트워크를 추가할 수 있는가?
docker 실행 시 자동으로 할당된 172.17.0.X가 아닌 static IP를 정의할 수 있는가 하는 의문이 들 수도 있다.
일단 docker0 대역에 있는 대역 IP는 staticIP를 할당할 수 없다. 때문에 staticIP를 사용하기 위해서는 새로운 user define network를 생성해주어야 한다.

```docker
$ docker network create --driver bridge \
--subnet 192.168.100.0/24 \ // 미설정 시 172.18.X.Y로 자동할당된다.
--gateway 192.168.100.254 \ // 미설정 시 192.168.100.1으로 자동할당된다.
my net

$ dockerk network ls
```


### Reference

https://m.youtube.com/watch?v=jOX80bXND2w

https://jangseongwoo.github.io/docker/docker_container_network/