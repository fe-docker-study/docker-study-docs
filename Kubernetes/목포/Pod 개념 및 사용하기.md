# Pod 개념 및 사용하기

## Pod란?

컨테이너를 표현하는 k8s API의 최소 단위
pod에는 하나 또는 여러 개의 컨테이너가 포함될 수 있음.

### pod 실행하기

1. CLI

```
$ kubectl run webserver --image=nginx:1.14
```

2. yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: webserver
spec:
	containers:
	- name: nginx-container
	  image: nginx:1.14
    imagePullPolicy: Always
	  ports:
	  - containerPort: 80
		 protocol: TCP
```

```
# pod 실행
$ kubectl create -f pod-nginx.yaml

# 현재 동작중인 pod 확인
$ kubectl get pods
$ kubectl get pods -o wide // 더 자세히 보기
$ kubectl get pods -o yaml // yaml 포맷으로 보기
$ kubectl get pods -o json // json 포맷으로 보기

# pod ip 정보만 확인하기
$ kubectl get pods webserver -o json|grep -i podip

# Pod에 접속해서 결과보기
$ curl <pod ' s IP address>
```

#### watch

2.0 초마다 해당 명령을 실행하는 작업

```
watch kubectl get pods -o wide
```

#### decribe

```
controlplane $ kubectl describe pod web1
Name:         web1
Namespace:    default
Priority:     0
Node:         node01/172.17.0.66
Start Time:   Sun, 20 Mar 2022 08:11:58 +0000
Labels:       run=web1
Annotations:  <none>
Status:       Running
IP:           10.244.1.2
IPs:
  IP:  10.244.1.2
Containers:
  web1:
    Container ID:   docker://e617e4314dabfa010659d8e770c979cdb2f8b5d3e36d3855bb1556dcd96292d8
    Image:          nginx:1.14
    Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 20 Mar 2022 08:12:15 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mv8c7 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-mv8c7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-mv8c7
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason                  Age                From               Message
  ----     ------                  ----               ----               -------
  Normal   Scheduled               25m                default-scheduler  Successfully assigned default/web1 to node01
  Warning  FailedCreatePodSandBox  25m                kubelet, node01    Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "6e4e86b6fc217d339c9f6244fbe3688bf832d9ec7bdca0b59aaa7c42396aad4a" network for pod "web1": networkPlugin cni failed to set up pod "web1_default" network: open /run/flannel/subnet.env: no such file or directory
  Warning  FailedCreatePodSandBox  25m                kubelet, node01    Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "defa766a7dc3ab8d28baba2c97353ea1fc92ffd0215fcfb526190fbcbf897852" network for pod "web1": networkPlugin cni failed to set up pod "web1_default" network: open /run/flannel/subnet.env: no such file or directory
  Normal   SandboxChanged          25m (x2 over 25m)  kubelet, node01    Pod sandbox changed, it will be killed and re-created.
  Normal   Pulling                 25m                kubelet, node01    Pulling image "nginx:1.14"
  Normal   Pulled                  25m                kubelet, node01    Successfully pulled image "nginx:1.14"
  Normal   Created                 25m                kubelet, node01    Created container web1
  Normal   Started                 25m                kubelet, node01    Started container web1
```

### multiful pods 생성하기

**pod-multi.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
  - name: centos-container
    image: centos:7
    command:
    - sleep
    - "10000"
```

```
$ kubectl create -f pod-multi.yaml


$ controlplane $ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
multipod    2/2     Running   0          46s   10.244.1.5   node01   <none>           <none>
web1        1/1     Running   0          28m   10.244.1.2   node01   <none>           <none>
webserver   1/1     Running   0          20m   10.244.1.4   node01   <none>           <none>
```

Ready를 보면 2개 중 2개의 컨테이너가 있고 하나의 IP를 가지고 있다.

```
# 두 개의 컨테이너 중 nginx에 접속하기
$ kubectl exec multipod -c nginx-container -it -- /bin/bash

# nginx 의 웹문서 바꿔보기
$ cd /usr/share/nginx/html
$ echo "TEST web" > index.html
exit
$ curl ip
TEST web

# 이번엔 centos 컨테이너에 접속해보기
$ kubectl exec multipod -c centos-container -it -- /bin/bash

# 여기서 80포트로 접속해보기
$ curl localhost
TEST web
```

어떻게 centos에 들어가서 80포트에 접속했는데 같은 결과가 뜨게되는것일까?
다른 컨테이너여도 같은 pod에 있기 때문에 (ip를 공유하기 때문에) 같은 결과를 표출하게 되는 것이다. (host name을 pod nmae으로 사용할 수 있다.)

### pod 내 컨테이너의 로그 보기

```
$ kubectl logs multipod -c nginx-container

172.17.0.65 - - [20/Mar/2022:08:42:42 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.58.0" "-"
172.17.0.65 - - [20/Mar/2022:08:47:34 +0000] "GET / HTTP/1.1" 200 9 "-" "curl/7.58.0" "-"
127.0.0.1 - - [20/Mar/2022:08:49:59 +0000] "GET / HTTP/1.1" 200 9 "-" "curl/7.29.0" "-"
```
