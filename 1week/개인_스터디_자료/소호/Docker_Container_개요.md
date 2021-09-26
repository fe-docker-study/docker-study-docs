# Docker, Container 의 개요



`이 글은 생활코딩(egoing)의 Docker 입문수업과 여러 블로그를 참고하여 정리한 자료입니다. (하단 참고자료 표기)`

----



# Container ?

**하나의 OS(host) 위에, 격리된 공간에서 프로세스가 동작하는 기술이다.**

![](https://images.velog.io/images/monadk/post/261cb476-b47b-4be6-8c47-b32a687d4da8/image-20210926180818379.png)

### 왜 사용하지 ?

기존의 가상화 기술은 주로 OS를 가상화하는 방식이었다. (VMware, VirtualBox...)

호스트 OS 위에 게스트 OS를 가상화 하는 것인데(HyperVisor 이용), 이는 운영체제 전체가 설치되어 무겁고 비효율적이기 때문에 운영 환경에서는 사용하기가 어렵다.

반면, 컨테이너는 AP 단위로 추상화한다. 

즉, 호스트 OS 커널은 공유하되 AP와 해당 AP에 필요한 바이너리, 라이브러리 등의 실행환경은 격리시키는 것이다.

별도의 가상 OS 없이 Host OS를 공유하기 때문에 시스템 부하가 적다는 점, 독립된 실행환경을 제공하기 때문에 AP 사이의 환경설정 충돌로 인한 문제나 별도의 서버 구축 비용 문제로부터 자유로울 수 있다는 장점이 있다.



![img](https://miro.medium.com/max/468/1*wDmCAYyj75DMvgI7Gu83zw.png)



컨테이너라는 기술은 **리눅스에 내장**되어 있는 것인데, 이로부터 아래 2가지 사실을 알 수 있다.

1. 컨테이너와 컨테이너 내에서 동작하는 AP들은 리눅스 OS에서 동작하는 AP라는 것.

2. Window, MacOS에서는 리눅스OS 가상머신을 설치하여 그 위에서 컨테이너가 실행되어야 한다는 것.

   → Docker를 설치하면 알아서 가상머신을 설치하고 리눅스를 설치하기 때문에 걱정하지 않아도 된다.

   → 다만, 가상머신 위에서 실행되기에 어느 정도의 <u>속도 저하</u>가 있다는 점을 감수해야 한다.

![](https://images.velog.io/images/monadk/post/3be725bc-8e48-48b9-8fcd-0fe4c1cfddec/image-20210926182143552.png)

---


# Docker ?

**컨테이너 기반의 오픈소스 가상화 플랫폼**  

즉, 위의 컨테이너 기술을 기반으로 하여 **컨테이너 생성과 관리, 빌드, 백업 및 배포**를 쉽게 만들어주는 오픈소스 소프트웨어이다. 

컨테이너 기술의 가벼움 + 도커가 제공하는 편리함이 이 기술을 흥하게 만든 게 아닐까 🤔

Docker에는 중요한 개념들이 있다. ↓ 

### 1) image

**컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 것**. 일종의 템플릿.

만약 Container를 프로세스라고 한다면, image는 프로그램의 개념이라고 보면 된다. image를 run하면 container가 생성되어 실행된다.

### 2) Docker hub (레지스트리)

이미지를 다운로드하고 배포할 수 있는(Pull/Push) 레지스트리 서비스들이 있는데, Docker에서 제공하는 기본 레지스트리가 Docker hub이다. (<https://hub.docker.com>)

Docker hub에서 image를 pull할 수 있고, 내가 생성한 image를 push 할 수도 있다. App store와 같은 개념이라고 보면 된다.

![](https://images.velog.io/images/monadk/post/08ddce49-62f3-4339-b6e9-c1769ed4f984/image-20210926221932056.png)







---

`※참고 자료※`

https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html

https://medium.com/@lhs6395/container%EC%99%80-vm%EC%9D%98-%EB%B9%84%EA%B5%90-84f6a8b7cd4c

https://www.youtube.com/watch?v=Ps8HDIAyPD0&list=PLuHgQVnccGMDeMJsGq2O-55Ymtx0IdKWf&index=1