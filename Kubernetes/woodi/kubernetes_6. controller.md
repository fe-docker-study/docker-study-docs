### 5장 내용 복습

- Pod API에 대해 다룸
- Pod : 컨테이너를 쿠버네티스 환경에서 실행할 수 있는 최소 단위 (실행 단위). 단, pod에는 2개 이상의 컨테이너가 포함될 수 있다.
- pod 생성, 운용 방법
    
    ![image](https://user-images.githubusercontent.com/47748246/160755274-b3d02b38-f94a-472a-b6fd-38685afd63f1.png)
    
- livenessProbe를 이용하여 건강한 pod만 실행하도록 하는 방법 (self-healing Pod)
- init container, infra container(pause) 이해
- static pod 만들기, Pod에 resource 할당하기
- 환경 변수를 이용해 컨테이너에 데이터 전달하기, pod 구성 패턴의 종류

![image](https://user-images.githubusercontent.com/47748246/160755292-9361f9f2-4d22-4f84-93f0-bc0cdc39445a.png)

# 6장 Controller

## 컨트롤러란?

- 오케스트라의 지휘자 역할
    - ex) 바이올린은 **몇 개**를 배치하고, 첼로는 몇 개를 배치하고 ~
- **Pod의 개수를 보장**
    
    ![image](https://user-images.githubusercontent.com/47748246/160755312-bfeec6bf-7485-48a3-ac41-612cc857af37.png)
    
- Controller 종류
    
    ![image](https://user-images.githubusercontent.com/47748246/160755333-792864e0-7668-485d-8cbd-8cb5e67f9d09.png)
    
    - Replicatoin Controller : 가장 basic한 컨트롤러
    - Replicaset : Replication Controller에서 부족한 점을 보완
    - Deployment : Replicaset의 부모 역할
    - Stateful sets : 상태를 보장해주는 컨트롤러
    - Job : batch job을 처리해주는 컨트롤러
    - Cronjob : batch job을 부모 입장에서 처리해주는 컨트롤러

## ReplicationController

- **요구하는 Pod의 개수를 보장**하며 파드 집합의 실행을 항상 안정적으로 유지하는 것을 목표
    - 현재 Pod의 개수 < 요구하는 Pod의 개수 → template을 이용해 Pod를 추가
    - 현재 Pod의 개수 > 요구하는 Pod의 개수 → 최근에 생성된 Pod를 삭제
- 기본 구성
    
    ![image](https://user-images.githubusercontent.com/47748246/160755357-6859bd6d-73c7-4689-9a69-d6a6afcb20ba.png)
    
    - 쿠버네티스야 나는 이런 (selector) key, value를 가지는  pod를 replicas개 운영해줘
    - 현재 운영되고 있는 pod들 중 위의 label selector에 해당하는 pod들의 개수 찾음
        - 적으면 → template을 이용해 pod 추가
        - 많으면 → 최근에 생성된 pod 삭제

### ReplicationController 동작 원리

![image](https://user-images.githubusercontent.com/47748246/160755399-39a1f9f0-d0a5-4b8d-9f09-3ab2c0dde1f0.png)

### ReplicationController-definition

![image](https://user-images.githubusercontent.com/47748246/160755421-02bdd066-4961-4468-ba1e-1133fb31cd3e.png)
- 실행
    
    `kubectl create -f rc-nginx.yaml`
    

## ReplicaSet

- ReplicationController와 같은 역할을 하는 컨트롤러
- ReplicationController 보다 **풍부한 selector 지원**
![image](https://user-images.githubusercontent.com/47748246/160755532-9cad3e5c-0bed-4e63-9edf-32982d4b2045.png)

- matchExpressions로 수식 사용
    - In : key와  values를 지정하여 key, value가 일치하는 Pod만 연결
    - NotIn : key는 일치하고 value는 일치하지 않는 Pod에 연결
    - Exists : key에 맞는 label의 pod를 연결
    - DoesNotExist : key와 다른 label의 pod를 연결
    
    예) `{key: tier, operator: In, values: [cache]}` :  tier 안에 cache가 존재해야함
    
    `{key: environment, operator: NotIn, values: [dev]}` : environment 안에 dev가 존재하지 않아야 함
    

### ReplicaSet definition

![image](https://user-images.githubusercontent.com/47748246/160755506-897c8165-f914-44f0-8c2d-b1dee36345e0.png)

## Deployment

- ReplicaSet을 컨트롤해서 Pod 수를 조절
- Rolling Update & Rolling Back
    - Rolling Update : pod 인스턴스를 점진적으로 업데이트하여 deployment 업데이트가 서비스 중단 없이 이루어질 수 있도록 해준다.
    
   ![image](https://user-images.githubusercontent.com/47748246/160755488-3e763fca-5858-4856-9673-4a71bd0edca6.png)
    

### Deployment definition

- kind만 빼고 ReplicaSet definition과 동일하게 생김
![image](https://user-images.githubusercontent.com/47748246/160755606-638742a5-6cd9-4b07-8feb-f7d8d8d5b72e.png)

### Deployment Rolling Update & Rolling Back

- Rolling Update
    - `kubectl set image deployment <deploy_name> <container_name> = <new_version_image>`

- RollBack
    - `kubectl rollout history deployment <deploy_name>`
    - `kubectl rollout undo deploy <deploy_name>`

![image](https://user-images.githubusercontent.com/47748246/160755657-8307de4b-b960-4afb-9e14-2c115b20fc21.png)
