# Controller
## 역할
어떤 애플리케이션을 몇 개 운영할 것인지 결정하고 Pod의 개수를 보장시켜준다.

## 종류
### Replication Controller

<img width="931" alt="image" src="https://user-images.githubusercontent.com/31172248/161410805-38601bd9-381a-4536-a439-7422058c093a.png">

1. Pod 생성 명령이 들어오면 ReplicationController를 하나 만든다.
2. Replica 개수를 확인하여 API는 Worker 노드들을 만든다.(요청한 Key:Value 값을 가진)
3. Controller는 노드들을 감시하고 있다가 Pod에 이상이 생길 시 조정해준다. 

Controller 중 가장 기본적인 역할을 하는 것으로 api 버전 1에서 만들어졌다. **요구하는 Pod의 개수를 보장하며 파드 집합의 실행을 항상 안정적으로 유지하는 것을 목표**로 한다. 
	* 요구하는 Pod개수가 부족하면 template을 이용해 Pod를 추가
	* 요구하는 Pod의 보다 많으면 최근에 생성된 Pod를 삭제

#### 기본구성(selector, replicas, template)
**yaml**
이런 key와 value를 가진 파드를 replicas 갯수대로 운영해달라는 의미. 만약 요구하는 Pod개수가 부족하면 컨테이너 템플릿을 이용해 이를 재생성한다. 
```yaml
.
.
spec:
  replicas: <배포갯수>
  selector: 
    key: value 
  template:
    <컨테이너 템플릿>
```

#### 예제1
**이 때 template의 labels는 미리 정의되어 있어야한다.**
rc-nginx.yaml
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
    app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
      spec:
        containers:
        - name: nginx-continaer
          image: nginx:1.14
```

pod의 이름 뒤에 붙은 건 임의의 해시값이다.
```console
$ kubectl create -f rc-nginx.yaml
$ kubectl get pods
NAME             READY   STATUS              RESTARTS   AGE
rc-nginx-njlds   0/1     ContainerCreating   0          6s
rc-nginx-ntmcm   0/1     ContainerCreating   0          6s
rc-nginx-p2vdc   0/1     ContainerCreating   0          6s

# 확인해보기
$ kubectl get replicationcontrollers 
# 또는
$ kubectl get rc
NAME       DESIRED   CURRENT   READY   AGE
rc-nginx   3         3         3       3m1s


# 자세히 보기
$ kubectl describe rc rc-nginx
Name:         rc-nginx
Namespace:    default
Selector:     app=webui
Labels:       app=webui
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=webui
  Containers:
   nginx-continaer:
    Image:        nginx:1.14
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  4m21s  replication-controller  Created pod: rc-nginx-p2vdc
  Normal  SuccessfulCreate  4m21s  replication-controller  Created pod: rc-nginx-njlds
  Normal  SuccessfulCreate  4m21s  replication-controller  Created pod: rc-nginx-ntmcm
```

#### 예제2
```console
# 대충 redis yaml 생성하기
$ kubectl run redis --image=redis --labels=app=webui --dry-run -o yredis.yaml

$ cat redis.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: webui
  name: redis
spec:
  containers:
  - image: redis
    name: redis


# 위 yaml 파일 create 시 pod가 생성되지 못하고 내려가버린다.
$ kubectl get pods -o wide
NAME             READY   STATUS              RESTARTS   AGE
rc-nginx-njlds   0/1     ContainerCreating   0          6s
rc-nginx-ntmcm   0/1     ContainerCreating   0          6s
rc-nginx-p2vdc   0/1     ContainerCreating   0          6s
redis  			  0/1     Treminating  		 	0          6s

.
.
몇 초 뒤
.
.

NAME             READY   STATUS              RESTARTS   AGE
rc-nginx-njlds   0/1     ContainerCreating   0          6s
rc-nginx-ntmcm   0/1     ContainerCreating   0          6s
rc-nginx-p2vdc   0/1     ContainerCreating   0          6s
```

이미 webui라는 라벨을 가진 컨테이너가 3개 존재하기 때문에 더 올릴 필요가 없는 것이다.

```console
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: ReplicationController
metadata:
  creationTimestamp: "2022-04-02T02:29:59Z"
  generation: 2
  labels:
    app: webui
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fif:metadata:
        f:labels:
          .: {}
          f:app: {}
      f:spec:
        f:replicas: {}
        f:selector:
          .: {}
          f:app: {}
        f:template:
          .: {}
          f:metadata:
            .: {}
            f:creationTimestamp: {}
            f:labels:
              .: {}
              f:app: {}
            f:name: {}
          f:spec:
            .: {}
            f:containers:
              .: {}
              k:{"name":"nginx-continaer"}:
                .: {}
                f:image: {}
                f:imagePullPolicy: {}
                f:name: {}
                f:resources: {}
                f:terminationMessagePath: {}
                f:terminationMessagePolicy: {}
            f:dnsPolicy: {}
            f:restartPolicy: {}
            f:schedulerName: {}
            f:securityContext: {}
            f:terminationGracePeriodSeconds: {}
    manager: kubectl
    operation: Update
    time: "2022-04-02T02:46:44Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:availableReplicas: {}
        f:fullyLabeledReplicas: {}
        f:observedGeneration: {}
        f:readyReplicas: {}
        f:replicas: {}
    manager: kube-controller-manager
    operation: Update
    time: "2022-04-02T02:46:46Z"
  name: rc-nginx
  namespace: default
  resourceVersion: "4128"
  selfLink: /api/v1/namespaces/default/replicationcontrollers/rc-nginx
  uid: 0299fd8f-2878-40ca-b369-0f0a2cf0e16b
spec:
  replicas: 4 
  selector:
    app: webui
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webui
      name: nginx-pod
    spec:
      containers:
      - image: nginx:1.14
        imagePullPolicy: IfNotPresent
        name: nginx-continaer
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 4
  fullyLabeledReplicas: 4
  observedGeneration: 2
  readyReplicas: 4
  replicas: 4


# 다시 실행
$ kubectl create -f redis.yaml
$ kbuectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
rc-nginx-5pb84   1/1     Running   0          5m45s   10.244.1.6   node01   <none>           <none>
rc-nginx-njlds   1/1     Running   0          22m     10.244.1.5   node01   <none>           <none>
rc-nginx-ntmcm   1/1     Running   0          22m     10.244.1.4   node01   <none>           <none>
rc-nginx-p2vdc   1/1     Running   0          22m     10.244.1.3   node01   <none>           <none>

# cli로 replicas 조절하기
# 가장 최근에 생성된 pod를 찾아 멈춘다.
$ kubectl scale rc rc-nginx --replicas=2
$ kubectl get pods -o wide
NAME             READY   STATUS        RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
rc-nginx-5pb84   0/1     Terminating   0          8m31s   <none>       node01   <none>           <none>
rc-nginx-njlds   0/1     Terminating   0          25m     10.244.1.5   node01   <none>           <none>
rc-nginx-ntmcm   1/1     Running       0          25m     10.244.1.4   node01   <none>           <none>
rc-nginx-p2vdc   1/1     Running       0          25m     10.244.1.3   node01   <none>           <none>
```

**Q. 만약 edit 모드에서 image 버전을 바꿨다면?**
Controller는 Selector만 보기 때문에 
**add-Q.그럼 만약 실행 중인 pod를 삭제했다면? (kubectl delete pod rc-nginx-vlcnb)**

이런 경우를 Rolling Update라고 한다. (무중단 배포) 이 경우에는 수동으로 했지만, 자동으로 수행하도록 설정해줄 수 있다????

### ReplicaSet Controller
Pod의 개수를 보장해주는 것은 Replication Controller와 동일하다. 하지만, 풍부한 Selector를 지원해준 다는 점에서 차이가 있다.(matcheLabels, matchexpressions..)

#### 비교하기
```console
# ReplicationController의 경우
# selector 조건이 여러 개일 시 해당 조건들이 "AND" 조건으로 들어가게 된다.
spec:
  replicas: 3
  selector:
    app: webui
    version: "2.1"
tempplate: . .

# ReplicaSet
# 아래와 같이 matchExpressions를 설정하면 버전은 저 중에 하나 여도 상관없음. 
spec:
  replicas: 3
  selector:
     matchLabels:
       app: webui
     matchExpressions:
     - {key: version, operator: In, value: ["2.1", "2.2"]}
     - {key: version, operator: NotIn, value: ["2.1", "2.2"]}
     - {key: version, operator: Exists}
template: . .
```


#### 기본구성
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    matadata:
      name: nginx-pod
      labels:
        app: webui
    spec: 
      containers:
      - name: nginx-container
        image: nginx:1.14
```
### 
* ReplicaSet의 selector
	* match Labels
		* key: value
	* matchExpressions
		* In: key와 Values를 지정하여 key, value가 일치하는 Pod만 연결
		* NotIn: key는 일치하고 value는 일치하지 않는 Pod에 연결
		* Exists: key에 맞는 label의 pod를 연결
		* DoesNotExist: key와 다른 label의 pod를 연결

#### 예제
```console
$ cat rs-nginx.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
$ kubectl create -f rs-nginx.yaml 
replicaset.apps/rs-nginx created

# --cascade=false 옵션을 추가하면 controller만 삭제하고 pod는 그대로 보존할 수 있다.
# 단독 pod로 바뀌게 된다. (Controller가 없어졌으니까)
$ kubectl delete rs rs-nginx
replicaset.apps "rs-nginx" deleted
$ kubectl get rs
No resources found in default namespace.
$ kubectl get pods --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
rs-nginx-9ktpp   1/1     Running   0          3m17s   app=webui
rs-nginx-b6vbd   1/1     Running   0          3m17s   app=webui
rs-nginx-tqx9j   1/1     Running   0          3m17s   app=webui
```


### Deployment
<img width="931" alt="image" src="https://user-images.githubusercontent.com/31172248/161410888-cd1ef4ff-1ed8-4757-8d6a-509ea8f337a1.png">

ReplicaSet을 제어하기 위한 부모 Controller이며 Rolling Update를 위해 만들어졌다. 

#### 비교하기
형식만 봤을 땐 kind 빼고는 똑같다. RollingUpdate의 지원 여부에 따라 차이가 날 뿐
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rs-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```

#### 기본구성
```console
$ cat deploy-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14


$ kubectl get rs,deploy,pod
NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/deploy-nginx-6f8645dc9   3         3         0       16s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deploy-nginx   0/3     3            0           16s

## naming : deploy-controller(replicaset)-podname
NAME                               READY   STATUS              RESTARTS   AGE
pod/deploy-nginx-6f8645dc9-7vgqf   0/1     Pending             0          16s
pod/deploy-nginx-6f8645dc9-9cwv8   0/1     Pending             0          16s
pod/deploy-nginx-6f8645dc9-gmdfh   0/1     ContainerCreating   0          16s

# ReplicaSet을 관리하는 것이 Deployment이기 때문에 이번엔 Controller를 지워도 Deployment가 다시 생성한다.(Pod까지 지워졌다가 또 다시 생성)
$ kubectl delete rs deploy-nginx-6f8645dc9
```


#### Rolling Update
<img width="931" alt="image" src="https://user-images.githubusercontent.com/31172248/161410909-84cbc323-9a28-42bf-b140-094ddd898b3b.png">


예) kubectl set image deployment <deploy_name> <container_name>=<new_version_image>

**Rollback**
kubectl rollout history deployment <deploy_name>
kubectl rollout undo deploy <deploy_name>

```console
$ kubectl set --help
Configure application resources

 These commands help you make changes to existing application resources.

Available Commands:
  env            Update environment variables on a pod template
  image          Update image of a pod template
  resources      Update resource requests/limits on objects with pod templates
  selector       Set the selector on a resource
  serviceaccount Update ServiceAccount of a resource
  subject        Update User, Group or ServiceAccount in a
RoleBinding/ClusterRoleBinding
```
살펴보니 env, image, resousrce.. 등을 update해줄 수 있다.

```console
# 기록하면서 현재 app-deploy에다가 app 컨테이너의 이미지를 nginx:1.15 버전으로 업데이트하라
kubectl set image deployment app-deploy app=nginx:1.15 --record

$ cat deployment-exam1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  selector:
    matchLabels:
      app: webui
  replicas: 3
  template:
    metadata:
      labels:
        app: webui
    spec:
      containers:
      - image: nginx:1.14
        name: web
        ports:
        - containerPort: 80

$ kubectl create -f deployment-exam1.yaml --record
$ kubectl set image deploy app-deploy web=nginx:1.15 --record
$ kubectl describe pod app-deploy-67777997f5-bhn87
Name:         app-deploy-67777997f5-bhn87
Namespace:    default
Priority:     0
Node:         node01/172.17.0.9
Start Time:   Sat, 02 Apr 2022 04:07:06 +0000
Labels:       app=webui
              pod-template-hash=67777997f5
Annotations:  <none>
Status:       Running
IP:           10.244.1.8
IPs:
  IP:           10.244.1.8
Controlled By:  ReplicaSet/app-deploy-67777997f5
Containers:
  web:
    Container ID:   docker://fb0d25d31fb7d76af6c18860f15434053a9348bff30d5a0de46bfe8c5def5f6f
    Image:          nginx:1.15
    Image ID:       docker-pullable://nginx@sha256:23b4dcdf0d34d4a129755fc6f52e1c6e23bb34ea011b315d87e193033bcd1b68
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 02 Apr 2022 04:07:09 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2dccr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-2dccr:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2dccr
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  61s   default-scheduler  Successfully assigned default/app-deploy-67777997f5-bhn87 to node01
  Normal  Pulled     60s   kubelet, node01    Container image "nginx:1.15" already present on machine
  Normal  Created    60s   kubelet, node01    Created container web
  Normal  Started    59s   kubelet, node01    Started container web


# rolling update 되는 기록을 보려면
$ kubectl rollout status deployment app-deploy 
Waiting for deployment "app-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "app-deploy" rollout to finish: 1 old replicas are pending termination...
deployment "app-deploy" successfully rolled out

# update 일시정지, 재시작
$ kubectl rollout pause deployment app-deploy
$ kubectl rollout resume deployment app-deploy

$ history 보기 (default limit = 10)
$ kubectl rollout history deployment app-deploy
deployment.apps/app-deploy 
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deployment-exam1.yaml --record=true
2         kubectl set image deploy app-deploy web=nginx:1.15 --record=true
3         kubectl set image deploy app-deploy web=nginx:1.16 --record=true
```

#### 기타 옵션
```yaml
spec:
  progressDeadlineSeconds: 600
  revisionHistoryLimit: 10
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  replicase: 3

# progressDeadlineSeconds -> 600초 동안 업데이트를 진행하지 못하면 방금 전 실행했던 업데이터를 취소시키겠다.
# revisionHistoryLimit -> 보존하고 있는 업데이트 기록 개수
# maxSurge -> RollingUpdate 시 pod개수의 25%(3개의 25퍼 = 0.75 반올림하면 1개/ (기존 개수 3) + 1 = 4)를 더한 만큼의 pod만 운영하겠다.
# maxUnavailable -> Terminate 되는 파드의 개수(백분률)
```

#### Rollback 하기
```console
$ kubectl rollout --help
Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc
  
  # Check the rollout status of a daemonset
  kubectl rollout status daemonset/foo

Available Commands:
  history     View rollout history
  pause       Mark the provided resource as paused
  restart     Restart a resource
  resume      Resume a paused resource
  status      Show the status of the rollout
  undo        Undo a previous rollout



# 현재 업데이트 기록
$ kubectl rollout history deployment app-deploy
deployment.apps/app-deploy 
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deployment-exam1.yaml --record=true
2         kubectl set image deploy app-deploy web=nginx:1.15 --record=true
3         kubectl set image deploy app-deploy web=nginx:1.16 --record=true


# 이전 버전으로 롤백해보기
$ kubectl rollout undo deployment app-deploy

# 1번 revision으로 돌아가기
$ kubectl rollout undo deployment app-deploy --to-revision=1

$ kubectl rollout history deployment app-deploy
deployment.apps/app-deploy 
REVISION  CHANGE-CAUSE
2         kubectl set image deploy app-deploy web=nginx:1.15 --record=true
3         kubectl set image deploy app-deploy web=nginx:1.16 --record=true
4         kubectl create --filename=deployment-exam1.yaml --record=true


# 1.14 버전으로 rollback 확인
$ kubectl describe pod app-deploy-7f6fffdcdd-vvw4t 
Name:         app-deploy-7f6fffdcdd-vvw4t
Namespace:    default
Priority:     0
Node:         node01/172.17.0.9
Start Time:   Sat, 02 Apr 2022 04:25:53 +0000
Labels:       app=webui
              pod-template-hash=7f6fffdcdd
Annotations:  <none>
Status:       Running
IP:           10.244.1.13
IPs:
  IP:           10.244.1.13
Controlled By:  ReplicaSet/app-deploy-7f6fffdcdd
Containers:
  web:
    Container ID:   docker://2ce1534e8a7b0b5c73d45f3f6797b2daad01efa33468dc3ade96be1c38745564
    Image:          nginx:1.14
    Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 02 Apr 2022 04:25:56 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-2dccr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-2dccr:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-2dccr
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  73s   default-scheduler  Successfully assigned default/app-deploy-7f6fffdcdd-vvw4t to node01
  Normal  Pulled     71s   kubelet, node01    Container image "nginx:1.14" already present on machine
  Normal  Created    71s   kubelet, node01    Created container web
  Normal  Started    70s   kubelet, node01    Started container web
```


### DaemonSet
노드 당 1개 Pod가 한 개씩 실행되도록 보장한다. master <- [요청]log-agnet를 daemonset으로 실행해줘 / node가 늘어나면 그 노드에도 자동으로 할당하게 된다. 

만약 node2에 pod가 이상이 생기면 얘가 없어질 때까지 기다렸다가 다시 생성한다. 이 타입은 보통 로그 수집기, 모니터링 agent들을 운용할 때 사용한다. CNI network, kubeproxy 등은 내부적으로 이미 daemonset 형태로 동작 중이다. 

DaemonSet버전도 Rolling Update를 마찬가지로 지원한다.

#### 기본구성
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
spec:
  selector:
    matchLabels:
      app: webui
  # replicas: 3
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - image: nginx:1.14
        name: nginx-container
```


### STATEFUL SETS
StatefulSet은 Pod의 이름과 볼륨을 보장해주는 컨트롤러이다. 

```console
$ cat rc-nginx.yaml

apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
    app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14

$ kubectl get pods
NAME             READY   STATUS              RESTARTS   AGE
rc-nginx-njlds   0/1     ContainerCreating   0          6s
rc-nginx-ntmcm   0/1     ContainerCreating   0          6s
rc-nginx-p2vdc   0/1     ContainerCreating   0          6s

```
RecplicationController 타입으로 Pod를 만들었다고 해보자. 그럼 각 Pod의 이름이 랜덤으로 설정 된다.(보장되지 않음)

StatefulSet에게 n개의 Pod를 실행해달라고 요청하면 이 컨트롤러는 각 Pod에 0~n의 index를 붙여 생성한다. 이렇게 운영되면 어떤 파드가 삭제될 파드인지 다음은 어떤 이름으로 생성이 될 것인지 알 수 있다.

#### 기본구성
StatefulSet에는 serviceName이라는 것이 들어간다. podManagementPolicy의 default값은 OrderdReady로 설정되어있어 index 순서대로 생성된다. Parallel은 병렬적(동시적)으로 생성하게된다.
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx
spec:
  replicas: 3
  serviceName: sf-service
#  podManagementPolicy: OrderedReady
  podManagementPolicy: Parallel
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```

```console
$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          7s    10.244.1.4   node01   <none>           <none>
sf-nginx-1   1/1     Running   0          7s    10.244.1.3   node01   <none>           <none>
sf-nginx-2   1/1     Running   0          7s    10.244.1.5   node01   <none>           <none>
```


#### SacleOut ScaleDown
SacleDown을 하게되면 가장 큰 번호부터 삭제된다.
```console
$ kubectl scale statefulset sf-nginx --replicas=5

$ kubectl get pods -o wide
NAME         READY   STATUS              RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running             0          72s   10.244.1.4   node01   <none>           <none>
sf-nginx-1   1/1     Running             0          72s   10.244.1.3   node01   <none>           <none>
sf-nginx-2   1/1     Running             0          72s   10.244.1.5   node01   <none>           <none>
sf-nginx-3   0/1     ContainerCreating   0          3s    <none>       node01   <none>           <none>
sf-nginx-4   0/1     ContainerCreating   0          3s    <none>       node01   <none>           <none>


$ kubectl scale statefulset sf-nginx --replicas=2

$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
sf-nginx-0   1/1     Running   0          2m11s   10.244.1.4   node01   <none>           <none>
sf-nginx-1   1/1     Running   0          2m11s   10.244.1.3   node01   <none>           <none>
```


### Job
```console
$ kubectl run testpod --image=centos:7 --command sleep 5

$ kubectl get pods --watch
NAME         READY   STATUS              RESTARTS   AGE
sf-nginx-0   1/1     Running             0          7m46s
sf-nginx-1   1/1     Running             0          7m46s
testpod      0/1     ContainerCreating   0          56s
testpod      1/1     Running             0          60s
testpod      0/1     Completed           0          65s
testpod      1/1     Running             1          66s
testpod      0/1     Completed           1          71s
testpod      0/1     CrashLoopBackOff    1          81s
testpod      1/1     Running             2          82s
testpod      0/1     Completed           2          87s          
```

Kubernetes는 기본적으로 Pod를 항상 Running 중인 상태로 유지 시키는게 목적이다. 때문에 Pod가 내려가도 계속 다시 실행시킨다.

하지만, 굳이 작업이 끝난 Pod를 재시작할 필요는 없다. 이런 배치 작업같은 경우는 ‘정상적으로 작업이 완료됐는지’의 여부를 받아 재시작을 할지 안 할지 정한다.

#### 기본구성
container가 만약 백업 작업, 가비지 데이터 클리어 작업, 로그 포워딩 작업 등의 배치성을 띠는 경우 사용하는 것이 적합하다.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
#  completions: 5 -> 아래 있는 컨테이너를 순차적으로 5번 실행
#  parallelism: 2 -> 동시에 실행 할 개수
  activeDeadlineSeconds: 5 -> 이 애플리케이션 적어도 5초 안에 안 끝나면 강제로 complete 시키겠다는 옵션
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 80; echo 'Bye'"
      restartPolicy: Never -> Pod를 Restart
#      restartPolicy: OnFailure -> Container만 Restart
#      backoffLimit: 3 -> 반복 횟수
```

만약 다음과 같이 pod를 실행하고 강제종료 시키면 처리해야할 작업이 다 끝나기전에 종료가 됐기 때문에 ‘비정상 종료’로 처리된다. -> 다시 pod를 실행
```console
$ kubectl create -f job-exam.yaml 
job.batch/centos-job created
controlplane $ kubectl get pods
NAME               READY   STATUS              RESTARTS   AGE
centos-job-zpwq9   0/1     ContainerCreating   0          3s
controlplane $ kubectl delete pod centos-job-zpwq9 
pod "centos-job-zpwq9" deleted
```

이렇게 다시 실행하고 제대로 작업이 완료된다고 해서 Job pod가 삭제되진 않는다. 만약 삭제되면 기존 처리했던 작업에 대한 로그나 목록을 볼 수 없다. 

**batch job만 삭제하기**
```console
$ kubectl delete job.batch centos-job
```



### CronJob
<img width="931" alt="image" src="https://user-images.githubusercontent.com/31172248/161410922-a5c177dd-267b-4829-bc5d-2f7829f666a0.png">

CronJob은 Job을 Control해서 원하는 시간에 작업을 실행할 수 있도록 제어한다.
job 컨트롤러로 실행할 Application Pod를 주기적으로 반복해서 실행 Linux의 cronjob 스케줄링 기능을 job controller에 추가한 API 다음과 같은 반복해서 실행하는 job을 운영해야 할 때 사용한다.

```
# Coronjob "0 3 1 * *"
# Minites(from 0 to 59)
# Hours(from 0 to 23)
# Day of the month(from 1 to 31)
# Month(from 1 to 12)
# Day of the week (from 0 to 6)
```


#### 기본구성
Job의 yaml 파일과 다른 점은 schedule빼고는 없다. job template을 그대로 가져다 쓰면 된다.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: centos-job
spec:
  schedule: "0 3 1 * *"
  jobTemplate:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 80; echo 'Bye'"
      restartPolicy: Never
```


#### 예제
```console
$ cat cronjob-exam.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-exam
spec:
  schedule: "* * * * *"
  startingDeadlineSeconds: 500 -> 500 초 안에 job이 실행되지 않으면 취소시킴
#  concurrencyPolicy: Allow -> default 한 번에 여러 작업이 여러 개 동작되도록 허용
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo Hello; sleep 80; echo Bye
          restartPolicy: Never

# 1분에 한 번씩 반복 실행
$ kubectl get pods -o wide
NAME                            READY   STATUS      RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
cronjob-exam-1648957680-mr49v   0/1     Completed   0          21s   10.244.1.4   node01   <none>           <none>



$ kubectl get pods -o wide
NAME                            READY   STATUS      RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
cronjob-exam-1648957680-mr49v   0/1     Completed   0          63s   10.244.1.4   node01   <none>           <none>
cronjob-exam-1648957740-ck68b   1/1     Running     0          3s    10.244.1.5   node01   <none>           <none>
```


```console
$ kubectl describe cronjobs.batch cronjob-exam 
.
.
Schedule:                      * * * * *
Concurrency Policy:            Forbid
Suspend:                       False
Successful Job History Limit:  3
.
.

```

완료된 기록은 최근 기록 중 n개만 남기도록 설정할 수도 있다.
