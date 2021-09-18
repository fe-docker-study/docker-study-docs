# Docker 개요

Docker는 컨테이너 기반의 오픈소스 가상화 플랫폼이라고 할 수 있다.

## 컨테이너? 그게 뭔가요

여기서 컨테이너란 리눅스 컨테이너를 말한다. 리눅스 컨테이너는 운영체제 수준의 가상화 기술로 **리눅스 커널을 격리시키고 공유**한다.

<br/>

### Vmware와 Docker의 비교

그렇다면 리눅스 커널을 공유하는 경우와 하지 않는 경우를 vmware와 비교하여 알아보자.

일단 설명하기전 운영체제의 개념을 다시 보자.
컴퓨터는 하드웨어와 소프트웨어로 이루어져있고 소프트웨어는 Application과 OS로 나눠진다.

좀 더 자세한 구조는 다음과 같다.

![os 구조](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4543d1af-bb0b-4f21-bc94-291fd4fb68d2/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210918%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210918T035248Z&X-Amz-Expires=86400&X-Amz-Signature=e6e2c9ede795274ddea034c6bdb3364e09f0eb3080e263254ecfe950141fccc4&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22 "사용자, 어플리케이션, OS, 하드웨어와의 관계")

(각 기능들이 어떤 역할을 하는지 궁금하시다면 추후 첨언하겠습니다.)

여기서 User~API는 User Space, System Call~CPU 레벨(Hadware) 까지는 Kernel Space로 영역으로 지칭된다. 이 커널은 OS와 용어를 혼용하여 쓰는 경우가 많은 듯하다.

자, 커널의 의미를 대충 알았으니 다시 Vmware와 리눅스 컨테이너와의 비교로 돌아오자.
![vmware와 리눅스 컨테이너 비교](https://images.velog.io/images/shinmj1207/post/d071e5c8-d6a8-4fa8-a6a8-512062df95d6/image.png)

**Vmware의 경우(왼)** 하드웨어 자원(Host OS) 자체를 가상화하여 자원을 공유하는 OS(Guest OS)를 관리한다. 총 2개의 커널을 가지게 되는 셈이다. 또, 이는 각 가상머신이 OS(커널)를 자체적으로 가지고 있는 것이기 때문에 시스템의 부하가 커진다는 단점이 있다.

하지만, **Docker의 경우(우)** OS 커널 자체를 가상화하여 리소스를 공유하기 때문에 하나의 커널(Host OS)만을 필요로하게 된다. 또한, Application 레벨에서는 각 Application을 격리해서 실행할 수 있다. 이렇게 하게되면 하나의 서버에 여러 컨테이너가 존재하게되고 각 컨테이너는 하나의 os(Guest os)만 가진채 서로 영향을 미치지 않고 독립적으로 실행할 수 있다. 이 역할을 Docker Engine이 하게되는 것이다.

![Docker의 어플리케이션 컨테이너화](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb2ei77%2Fbtqz8m9yq5P%2Fv4xuv1ZNy10KoTQyQka7AK%2Fimg.png)

<details>
<summary style="font-weight:bold;">공유 방식 간단 설명(참고)</summary>
<div markdown="1">

예를 들어 Host OS가 Ubuntu veresion X고, Container의 OS가 CentOS version Y라고 했을 때 Conatiner에는 CentOS version Y의 full image가 들어있는 것이 아니라 Ubuntu versionX와 CentOS version Y의 차이가 되는 부분만 패키징 된다. Container내에서 명령어를 수행하면 실제로는 Host OS에서 그 명령어가 수행된다. 즉, Host OS의 Process 공간을 공유하게 되는 것이다.
(출처 : https://bcho.tistory.com/805)

</div>
</details>
<br/>
<br/>

## Docker Image 와 Docker File

### Image

Docker Image는 **컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 것**으로 상태값을 가지지 않고 변하지 않는다. 컨테이너는 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장된다. (컨테이너 삭제 시 복구 불가능)

이 이미지는 컨테이너 실행에 대한 모든 정보를 가지고 있기 때문에 용량이 크다. 이런 문제를 해결하기 위해 Layer라는 개념을 사용하고 유니온 파일 시스템을 이용해 여러 개의 레이어를 하나의 파일 시스템으로 사용할 수 있게 해준다.
이미지는 여러 개의 읽기 전용 read only 레이어로 구성되고 <u>파일이 추가되거나 수정되면 새로운 레이어가 생성</u>된다.
먼저 Container Image를 패키징 하기 위해서는 **Base Image**가 필요하다. 이는 기본적인 인스톨 이미지 이며, **Docker File**이라는 것을 통해 추가로 설치되는 스크립트를 정의할 수 있다.

예를 들어, Base Image인 ubuntu 이미지가 A + B + C의 집합이라면 Docker File을 통해 ubuntu + nginx(A + B + C + nginx) 이미지를 만들 수 있다. 또 webapp 이미지를 nginx 이미지 기반으로 만들었다면 A + B + C + nginx + source 레이어로 구성된다.

### Docker File

컨테이너에 설치해야하는 패키지, 소스코드, 명령어, 환경변수설정 등을 기록한 하나의 파일이다. 그리고 이를 빌드하면 자동으로 이미지가 생성된다.

#### 예시

```DockerFile
# Step1
FROM centos:7

# Step2
RUN touch /etc/yum.repos.d/nginx.repo && echo -e '[nginx]\nname=nage repo\nbaseurl=http://nginx.org/packages/centos/7/$basearch/\ngpgcheck=0\nenabled=1' > /etc/yum.repos.d/nginx.repo

# Step3
RUN yum -y install nginx

# Step4
EXPOSE 80

# Step5
CMD ["nginx", "-g", "daemon off;"]
```

Image 섹션에서 설명했던 것과 같이 Image Layer의 구조대로 커맨드를 적어 build하게 되면 스탭별로 이미지가 생성되게 된다. (step1 -> step5)

<br/>
<br/>
<details>
<summary style="font-weight:bold;">Image 경로(참고)</summary>
<div markdown="1">

![도커 이미지 경로](https://images.velog.io/images/shinmj1207/post/0c96cf24-425e-4af4-9e00-270a8238f244/image.png)

이미지는 url 방식으로 관리하며 태그를 붙일 수 있다. docker.io/library 는 생략가능하다.

</div>
</details>
<br/>
<br/>

## Docker Registry

우리는 위에서 만든 이미지를 Resgistry에 저장했다가 다른 환경에서 가져다가 사용할 수도 있다.
서버 이전을하거나 새로운 환경을 구축할 때 굳이 모든 설정을 다시 할 필요가 없이 이미지만 실행하면 된다는 것.
저장소에 저장하거나 받아오는 방식은 Git과 유사하다.

## Docker Compose

여러 컨테이너의 실행을 한 번에 관리할 수 있도록 해주는 것. 설명만 봐서는 잘 안 와닿을 것 같아 다음 스터디 때 실습과 더불어 자세하게 기술할 예정.

<br/>
<br/>
---
<details>
<summary>[출처]</summary>
<div markdown="2">

https://bcho.tistory.com/805

https://cultivo-hy.github.io/docker/image/usage/2019/03/14/Docker정리/

https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html

</div>
</details>
