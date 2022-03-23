## static Pod (kubelet daemon)

**지금까지의 Pod**

- 쿠버네티스가 kubectl 명령으로 pod를 만들어 달라고 명령하면(= master의 API에게 전달하면), master API는 etcd 정보를 가져다가 scheduler에게 전해준다.
- Scheduler는 etcd 정보를 바탕으로 어떤 slave Node에게 줄지 결정하고 API에게 응답
- API는 pod를 결정된 node에게 전달, 실행 시킴

**static Pod**

- master API에게 요청을 보내지 않음
- slave node들에는 각각 kubelet daemon이 실행 중
- 이 kubelet이 관리하는 static pod 디렉토리(`/etc/kubernetes/manifests/`)에 yaml 파일을 저장하면 알아서 POD를 실행함. (반대로 삭제하면 pod도 사라짐)
- 다시 말해, API 서버 없이 특정 노드에 있는 kubelet 데몬에 의해 직접 관리되는 pod를 static pod라 한다.

## Pod에 리소스(cpu, memory) 할당(제한)하기

- pod를 실행할 때 발생할 수 있는 문제점
    1. Scheduler가 해당 pod를 실행할 리소스가 충분하지 않은 node로 pod를 보낼 수 있음
    2. 한 pod가 node의 전체 Resource를 다 잡아먹어서 그 node에 있는 다른 pod가 실행이 안될 수 있음
- 이를 방지하기 위해
    1. pod에 이 pod를 실행하기 위한 리소스를 명시해준다. **[request]**
        - pod를 실행하기 위한 최소 리소스 양을 요청
        - ex. pod : cpu는 500mc 이상이고, memory는 100MB 이상의 리소스 여유가 있는 node로 보내줘
    2. pod에 이 pod에 할당될 수 있는 resource의 최대 값을 세팅해줌 **[limit]**
        - pod가 사용할 수 있는 최대 리소스 양을 제한
        - Memory limit을 초과해서 사용되는 pod는 종료 (OOM Kill)되며 다시 스케줄링 된다.

![image](https://user-images.githubusercontent.com/47748246/159627974-eb22ab21-663e-4601-a0a4-dddd53655714.png)

- 단위
    - cpu : m (밀리코어) or 코어 개수
        - 코어 1개 = 1000m
    - memory : Mi (milli bytes)

## 쿠버네티스 Pod 환경변수 설정과 실행 패턴

### 환경 변수

- Pod 내의 컨테이너가 실행될 때 필요로 하는 변수
- 컨테이너 제작 시 미리 정의
    - ENV NGINX_VERSION 1.19.2
    - ENV NJS_VERSION 0.4.3
- Pod 실행 시 미리 정의된 컨테이너 환경변수를 변경할 수 있다.

### 환경 변수 사용 예

![image](https://user-images.githubusercontent.com/47748246/159627998-c3d32c55-9df5-4a91-ac79-e54d29bd6625.png)

### Pod 구성 패턴의 종류

- Pod 실행 패턴
    - pod를 구성하고 실행하는 패턴
    - multi-container Pod
        
        ![image](https://user-images.githubusercontent.com/47748246/159628020-33c64724-adc4-4d6b-97c1-804f18e85551.png)
        
        - Sidecar
            - 하나의 Pod 안에 2개의 컨테이너 존재.
            - 한 컨테이너(예를 들면, 웹 서버)가 log 파일을 생성해서 저장.
            - 다른 컨테이너 (Sidecar)가 log 파일을 읽어서 분석함
            - 즉, 한 컨테이너가 실행되면서 결과물을 만들어야만, 다른 컨테이너가 그 결과물로 무언가를 할 수 있는 형태
        - Adapter
            - 현재 시스템의 상태를 나타내는 외부 모니터링 서비스 존재
            - Adapter가 이 외부 모니터링 시스템으로 부터 정보를 가져와서  app container에게 전달
            - App container는 일종의 UI 시스템. Adapter로부터 받은 monitoring 정보를 사용자에게 보여 줌
        - Ambassador
            - App container에서 데이터를 생산
            - Ambassador 컨테이너가 로드밸런싱 역할. 외부의 다른 DB/캐시로 데이터 보냄
