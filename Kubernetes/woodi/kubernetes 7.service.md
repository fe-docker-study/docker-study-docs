<aside>
📖 서비스 = 쿠버네티스 네트워크

</aside>

## 1. Kubernetes Service 개념

- 동일한 서비스를 제공하는 **Pod 그룹의 단일 진입점**을 제공
    
    ![image](https://user-images.githubusercontent.com/47748246/162397131-e960bd02-0e33-4a9a-b933-a68c226191b6.png)
    

### Service definition

![image](https://user-images.githubusercontent.com/47748246/162397167-945682a9-f88a-4358-af82-b7a5949f38bc.png)
## 2. Kubernetes Service 타입

- 4가지 Type 지원
    - ClusterIP (default)
        - Pod 그룹의 **단일 진입점(Virtual IP(=loadbalancer ip))** 생성
    - NodePort
        - 기본적으로 ClusterIP가 생성됨
        - 추가로, 모든 Worker Node에 외부에서 접속 가능한 포트가 open
    - LoadBalancer
        - NodePort + LB 장비
            - 실제 LB 장비의 ip address와 NodePort를 연결시켜줌
        - 클라우드 인프라스트럽처(AWS, Azure, GCP 등)나 오픈 스택 클라우드에 적용
        - LoadBalancer를 자동으로 프로 비전하는 기능 지원
    - ExternalName
        - 클러스터 안에서 외부에 접속 시 사용할 **도메인을 등록**해서 사용
        - 클러스터 도메인이 실제 외부 도메인으로 치환되어 동작

### ClusterIP

- 쿠버네티스에게 deploy를 통해서 요청
    - “웹 서버를 webui라는 이름으로 3개 운영해줘”
    - 이때 생성된 webui 파드들은 **각자 서로 다른  ip를 가짐**
- 이처럼 selector의 label이 동일한 파드들을 그룹으로 묶어주기 위해 service API가 필요하다.
- 즉, Service API를 통해 **단일 진입점 (Virtual IP)를 생성**하여 동일한 파드들(label로 동일한지 여부를 확인)을 로드밸런싱한다.
- type 생략 시 default 값으로 10.06.0.0/12 범위에서 할당됨

예제

![image](https://user-images.githubusercontent.com/47748246/162397250-8e8a131d-ab03-4116-bff1-ea89fe748b19.png)

```json
$ kubectl create -f clusterip-nginx.yaml
$ kubectl get svc

$ curl 10.100.100.100
$ kubectl describe svc clusterip-service
$ kubectl delete svc clusterip-service
```

### NodePort

- 모든 노드를 대상으로 **외부 접속 가능한 포트를 예약**
- Default NodePort 범위 : 30000 - 32767
- ClusterIP를 생성한 후 NodePort를 예약

예제

![image](https://user-images.githubusercontent.com/47748246/162397311-399ae37d-3d73-4826-96a2-4316320c9021.png)

### LoadBalancer

- LoadBalancer는 Public 클라우드 (AWS, Azure, GCP 등)에서 운영 가능
- LoadBalancer를 자동으로 구성 요청
- NodePort를 예약 후 해당 nodeport로 외부 접근을 허용

![image](https://user-images.githubusercontent.com/47748246/162397355-69f34232-596e-4750-a42d-6c3c6a1ec567.png)

![image](https://user-images.githubusercontent.com/47748246/162397398-029f159f-cadf-40d1-8088-c29fdd994bf9.png)

### ExternalName

- 클러스터 내부에서 External(외부)의 도메인을 설정

![image](https://user-images.githubusercontent.com/47748246/162397438-498f3530-3b7d-48d7-8e41-9cbe826708ac.png)

![image](https://user-images.githubusercontent.com/47748246/162397479-8e7c1925-e49d-4ff1-9d28-967fb09a19ca.png)

## 3. Headless Service

- ClusterIP가 없는 서비스. 단일 진입점이 필요 없을 때 사용
    - endpoint를 묶어주긴 하지만 그에 대한 ip address는 없음
- Service와 연결된 Pod의 endpoint에 대한 DNS 레코드가 core DNS에 생성됨
    
    → pod에 대한 endpoint를 DNS resolving 서비스로 요청할 수 있음
    
- Pod의 DNS 주소 : pod-ip-addr.namespace.pod.cluster.local

![image](https://user-images.githubusercontent.com/47748246/162397524-660dcdcf-3c26-47f9-a600-41ebc1787e20.png)

## 4. kube-proxy

- Kubernetes Service의 backend 구현
- endpoint 연결을 위한 iptables 구성
- nodePort로의 접근과 pod 연결을 구현 (iptables 구성)

### Kube-proxy mode

- userspace
    - 클라이언트의 서비스 요청을 iptables를 거쳐 kube-proxy가 받아서 연결
    - kubernetes 초기 버전에 잠깐 사용
- iptables
    - defualt kubernetes network mode
    - kube-proxy는 service API 요청 시 iptables rule이 생성
    - 클라이언트 연결은 kube-proxy가 받아서 iptables 룰을 통해 연결
- IPVS
    - 리눅스 커널이 지원하는 L4 로드밸런싱 기술을 이용
    - 별도의 ipvs 지원 모듈을 설정한 후 적용 가능
    - 지원 알고리즘 : rr(round-robin), lc(least connection), dh (destination hashing), sh(source hashing), sed(shortest expected delay), nc
