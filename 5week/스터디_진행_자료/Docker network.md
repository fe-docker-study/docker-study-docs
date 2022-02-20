# Docker network

## 직접 Host 네트워크를 만들어보자.. 

```
sudo docker pull nockchun/docterm
```

- Docker network의 실행 상태 및 작동 내용을 확인하기 위한 패키지

- Pull 받은 뒤, 이미지가 잘 내려왔는지 아래 명령어로 확인할 수 있습니다.

```
sudo docker images
```

- 내려받은 이미지를 이용해 컨테이너를 생성해줍니다.

```
sudo docker run -d --net host --name imhost nockchun/docterm
```

-  ` --net ` : 네트워크의 종류 지정 (host, bridge, none, overlay)
- ` --name ` : 컨테이너의 이름 지정
- `docker ps` 명령어로 잘 올라왔는지 확인합니다.

```
sudo docker exec -it imhost bash
```

- 위 명령어를 통해 내가 만든 `imhost`라는 host network bash에 접속해봅시다
- `-it` 명령어는 bash에 접속해서 수정, 지속적인 연결을 할 수 있게 해줍니다.

```
bash-5.0# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:94:7d:28:ba:ec brd ff:ff:ff:ff:ff:ff
    inet 172.31.34.163/20 brd 172.31.47.255 scope global dynamic eth0
       valid_lft 2411sec preferred_lft 2411sec
    inet6 fe80::894:7dff:fe28:baec/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:d8:01:c2:31 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d8ff:fe01:c231/64 scope link
       valid_lft forever preferred_lft forever
bash-5.0#
```

- `ip a` 명령어를 이용해 ip address 정보를 확인해봅시다..
- 중간에 보면, 172.31.34.163 으로 물리적 컴퓨터와 같은 네트워크를 공유하고 있는 것을 알 수 있습니다.

---

### 결론 

Host 네트워크를 사용한다는 것은, 물리적 컴퓨터와 네트워크를 공유하겠다는 의미이다.

실무에서 잘 사용되지 않는다.

---

## 직접 Bridge 네트워크를 만들어보자..

```
sudo docker network create mybridge
```

- `--net`설정을 하지 않는다면 기본 네트워크인 브릿지 네트워크가 생성됩니다.

- 해당 네트워크가 잘 생겼는지 확인해봅시다.
- 네트워크도 도커의 리소스로서 관리되기 때문에 아래의 명령어로 확인할 수 있습니다.

```
sudo docker network ls
```

| **NETWORK ID**   | **NAME** | **DRIVER** | **SCOPE** |
| ---------------- | -------- | ---------- | --------- |
| **5e10196c8582** | bridge   | bridge     | local     |
| **e9cf4c38cbdc** | host     | host       | local     |
| **9dd501d1e0ec** | mybridge | bridge     | local     |
| **0daa32ed6c6c** | none     | null       | local     |

- 이번엔 브릿지 네트워크 내에 컨테이너를 생성해봅시다.

```
sudo docker run -d --net mybridge --name mybridge1 nockchun/docterm
sudo docker run -d --net mybridge --name mybridge2 nockchun/docterm
```

- `mybridge`라는 네트워크에 아까 받은 이미지를 이용해  `mybridge1`,`2`라는 컨테이너를 생성했습니다.

```
sudo docker run -d --name bridge1 nockchun/docterm
sudo docker run -d --name bridge2 nockchun/docterm
```

- `--net`명령어 없이 사용하면, 자동으로 기본 브릿지 네트워크인 `docker0`에 해당 컨테이너가 할당됩니다.

---

#### 지금까지 기본 브릿지 네트워크와 직접 만든 브릿지 네트워크 내에 컨테이너를 생성했습니다.

## 뭐가 다른가요..?

- 브릿지는 컨테이너간의 라우터인 셈입니다.
- 같은 브릿지를 공유하는 컨테이너 간에는 네트워크 통신이 가능합니다.
- 다른 브릿지를 가진 컨테이너 간에는 네트워크 통신이 불가능합니다.
- 모든 브릿지는 물리적 네트워크 인터페이스에 연결이 되어있습니다.

## 확인해봅시다.

```
sudo docker exec -it bridge1 bash
```

- bridge1의 배쉬에 접근할 수 있는 명령어입니다.

- 해당 배쉬 내에서 `ip a`를 입력해봅니다.

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

- 위와 비슷한 내용이 뜹니다.
- 해당 컨테이너가 사용하는 ip주소는 `172.17.0.2` 임을 확인 할 수 있습니다.

```shell
sudo docker exec -it bridge2 bash
```

- 마찬가지로 bridge2 컨테이너의 bash에 접속해봅니다.
- `ip a`를 입력합니다.

```bash
bash-5.0# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

- 해당 컨테이너는 `172.17.03` 를 사용하고있음을 확인할 수 있습니다.

---

##### 두 컨테이너는 같은 브릿지를 공유하고 있기 때문에, 서로간의 통신이 가능합니다.

- 둘 중 하나의 bash에서 아래와 같이 ping을 찍어봅시다.

```bash
ping 172.17.0.2 // ping을 보낼 주소 찍으면됩니다. 다를 수 있음. 확인하고 보내세요~
```

- 주소는 핑을 보낼 ip주소를 찍으면 됩니다.

```bash
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.060 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=6 ttl=64 time=0.060 ms
64 bytes from 172.17.0.2: icmp_seq=7 ttl=64 time=0.060 ms
64 bytes from 172.17.0.2: icmp_seq=8 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=9 ttl=64 time=0.061 ms
```

- ping이 잘 가는걸 확인할 수 있습니다.

### 결론

- 정말 같은 브릿지를 공유하는 네트워크 간에는 통신이 가능하다.

---

#### 이번에는 다른 브릿지 네트워크의 컨테이너에게 핑을 보내봅시다...

- 당연히 안되지만..
- 아래 명령어로 `mybridge` 네트워크의 컨테이너에 접속합니다.

```
sudo docker exec -it mybridge1 bash
```

- `ip a` 명령어를 입력해 ip를 확인해봅시다.

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
11: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

- `172.18.0.2`로 아까 docker0 네트워크의 서브넷과 다른 서브넷을 사용함을 알 수 있습니다.
- 이번엔 여기서 `bridge1` 컨테이너에 핑을 보내봅시다..
- `ping 172.17.0.2` 명령어로 핑을 찍으면

```bash
bash-5.0# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
```

- 핑이 안나갑니다... 네트워크 통신이 안되고 있습니다. 

### 결론

- 같은 브릿지가 아니라면 네트워크 통신이 안된다.

---

## 마지막으로 

- 그러나 모든 브릿지 네트워크는 HOST 에서 접속이 가능합니다.
- 라우터들이 모두 물리적 네트워크 인터페이스에 연결되어있기 때문입니다.
- 이번엔 host에서 두 브릿지에 ping을 보내봅시다.
- 놀랍게도 다 잘 됩니다.
- 또, 꼭 ip주소로 입력하지 않고 해당 컨테이너의 이름으로 핑을 보내도 잘 됩니다.
- 단 docker0의 컨테이너들은 안됩니다.