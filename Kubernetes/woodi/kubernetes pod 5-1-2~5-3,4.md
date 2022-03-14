## Pod 동작 flow

![image](https://user-images.githubusercontent.com/47748246/158134706-a1d3cf4e-0082-49ec-8add-ff655d231fcb.png)

- `kubectl run webserver —image=nginx:1.14` 와 같은 명령을 쿠버네티스에 내렸을 때
- API는 해당 명령이 pod 실행 문법에 일치하는지 확인한다.
- 그리고 etcd를 통해 worker Node의 상태 정보를 받아와서 scheduler에게 어느 노드를 사용해야할지 물어본다.
- Scheduler는 etcd 정보를 통해 어떤 Node로 pod를 보낼지 결정하고 api에게 응답한다.
- 그때까지 pod의 상태는 `Pending` 상태이다.
- 그리고 마침내 pod가 node에 배치를 받고 실행되면, `Running` 상태가 된다. 혹은 running을 못하고 실패할 경우 `Failed` 상태가 될 수 있다.

### Pod 관리하기

- 동작 중인 pod 정보 보기
    
    ```bash
    $ kubectl get pods
    $ kubectl get pods -o wide
    $ kubectl describe pod webserver
    ```
    

- 동작 중인 pod 수정
    
    ```bash
    $ kubectl edit pod webserver
    ```
    

- 동작 중인 pod 삭제
    
    ```bash
    $ kubectl delete pod webserver
    $ kubectl delete pod --all
    ```
    

## Pod - livenessProbe를 이용해서 Self-healing Pod 만들기

(kubelet으로 컨테이너 진단하기)

### Self-healing이란

- 컨테이너가 제대로 동작되지 못할 때  restart 해주는 기능
- 즉, 건강한 컨테이너로 애플리케이션이 동작되는 걸 보장해주는 기능
- self-healing 기능 안에 포함된 것이 livenessProbe 기능임

### 그냥 pod와 libenessProbe 기능이 추가된 pod의 definition 차이

![image](https://user-images.githubusercontent.com/47748246/158134763-46363cce-d87e-41fc-ab72-100f51ce3b69.png)

- 위 예제에서 컨테이너는 웹 서비스이기 때문에 livenessProbe하기 위해 http 프로토콜로 80포트에 요청하여 health check한다.

### livenessProbe 매커니즘

- 컨테이너의 종류에 따라서 health check하는 방법이 다르다.

![image](https://user-images.githubusercontent.com/47748246/158134803-67f65357-87bb-4fab-8e80-c65c762ab522.png)

- 단, 여기서 주의할 것은 restart 시키는 대상이 pod가 아니라 container라는 것이다.
- 그렇기 때문에 restart를 시키더라도 pod에 할당된 ip 주소는 변하지 않는다.

## Pod - init container & infra container

[https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)

### initContainer란?

- main container를 실행하는데 필요한 환경 세팅 및 초기화 구성을 지원해주는 컨테이너
- 예를 들어서 main container가 node js 앱의 login 기능을 하는 컨테이너라 하자. login을 위해서는 우선적으로 db를 조회해서 로그인 관련 정보들을 가져오는 컨테이너가 필요한데 이를 init container라 할 수 있다 .
- init container가 성공해야만 main container를 실행할 수 있다.
- 참고) 하나의 pod 안에 init container + main container가 함께 있음.

init-container-exam.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

- 위 pod 처럼 initContainer가 2개일 경우, 2개 다 성공해야 mainContainer가 동작한다.
- 만약 쿠버네티스 내에 intiContainer가 실행 중이지 않다면 mainContainer는 실행되지 않는다.
- 따라서 별도로 init-service와 init-mydb 컨테이너를 실행시켜줘야 myapp-container가 동작하게 된다.

### InfraContainer (pause)란?

- 컨테이너를 포함하는 Pod를 만들 때, 그 pod 안에 pause라는 컨테이너도 함께 생성된다.
- 이 pause 컨테이너는 해당 Pod에 대한 infra 정보 (ip, host name 등)을 관리하고 생성해준다.
![image](https://user-images.githubusercontent.com/47748246/158134845-f719c283-d0de-4632-aae4-bac5d7cd5bae.png)
