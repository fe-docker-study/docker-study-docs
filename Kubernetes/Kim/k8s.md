---
layout: post
title: "4-1 ~ 5-1-1"
---

# 쿠버네티스 동작원리
![image](https://user-images.githubusercontent.com/86642180/158152042-556110a5-3c2a-428e-a269-2a1567778da8.png)
1. 도커에서 컨테이너 생성  
2. 도커 push를 이용해서 Docker hub에 저장  
3. cli 혹은 yaml 파일로 kunctl 명령어 생성  
4. control plane으로 컨테이너 저장  
5. control plane에서 스케줄러를 통해 어떤 노드에서 실행하는것이 효율적일지 판단 뒤 컨테이너 실행요청  

<br>

# 쿠버네티스 아키텍처
![image](https://user-images.githubusercontent.com/86642180/158159437-ec1e784a-d847-4638-b5b6-0f4d42873ed5.png)
사실 100% 이해된건 아니라 나중에 한번 더 봐야겠다..  
이전의 동작원리를 참고해서 보자면  
`kubectl create ngix ~~~`  
👉 control plane의 API가 요청을 받아들임  
👉 각자 워커노드들에 있는 kubelet을 통해 control plane의 etcd 저장소에 워커노드들의 상태를 확인, 문법 체크  
👉 스케줄러에서 어떤 노드가 가장 효율적으로 작동할지 판단 & API에 정보 전달  
👉 API가 kubelet을 통해 선택된 워크노드에게 요청(docker ngix를 통해)  
👉 도커는 도커플랫폼에서 원하는 엔진을 허브에서 받아온 뒤 작동  

<br>

# namespace
K8s의 API 중 하나의 종류  
클러스터 하나를 여러 개의 논리적인 단위로 나눠서 사용  

<br>

<i>생각해보니 스프링 namespace 정의도 말해보라 하면 모르겠다..  
매퍼에서 namespace를 통해 sql문 나누고 동작을 따로하게 만들었던거 정도?</i>

# yaml 템플릿
파이썬처럼 들여쓰기가 중요함  

<br>

# Pod
이전에 도커에서 container에 애플리케이션을 넣어 수행했으나  
쿠8에서는 Pos를 통해 컨테이너를 실행시킨다  
즉 Pod는 K8s API의 최소 단위다  
- pod yaml을 이용해 pod 실행!
```
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
    - name: nginx-container
      image: nginx: 1.14
      imagePullPolicy: Always
      ports:
      - containerPort: 80
        protocol: TCP
```
```
$ kubectl create -f pod-nginx.yaml
$ curl <pod's IP address
```

<br>
앞으로는 실행할 때 캡쳐도 하기...
