### yaml 템플릿과 API

- 사람이 쉽게 읽을 수 있는 데이터 직렬화 양식
- 기본 문법
    
    ```bash
    apiVersion: v1
    kind: Pod
    	child: first child
    	key2:
    		child-1: kim
    	key3:
    		- grandchild1:
    			name: kim
    		- grandchild2:
    			name: lee
    ```
    
- key : value 형식으로 데이터를 표현
- 들여쓰기를 할 때에는 tab이 아닌 space bar를 사용
- scalar 문법 : ‘:’을 기준으로 key:value를 설정
- 배열 문법 : ‘-’ 문자로 여러 개를 나열

**API version**

- alpha → beta → stable 순으로 발표
- k8s object 정의 시 apiVersion이 필요
    
    ```bash
    apiVersion: v1
    kind: Pod
    metaData:
    	name: mypod
    spec:
    	containers:
    	- image: nginx:1.14
    		name: nginx
    		ports:
    		- containerPort: 80
    		- containerPort: 443
    ```
    
- k8s가 update하는 API가 있으면 새로운 API가 생성됨
- API Object의 종류 및 버전
    - Deployment        apps/v1
    - Pod                      v1
    - ReplicaSet          apps/v1
    - ReplicationController       v1
    - Service                v1
    - PersistentVolume     v1

- 만약 오브젝트를 정의할 때 해당 api의 버전을 잘 못 명시하면 오류 발생한다.
- 위의 모든 오브젝트의 버전을 알고 있을 필요 x,
- kubectl explain pod   명령을 통해 해당 api의 버전을 확인할 수 있다.
