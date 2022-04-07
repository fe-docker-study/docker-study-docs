#Kubernetes/Kubernetes-Service

# 쿠버네티스 Service

## 서비스 개념

## 서비스 Definition

```console
apiVersion: v1
kind: Service
metadata:
  name: webui-svc
spec:
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80 // cluter ip 의 port
    targetPort: 80 // 각 pod들의 port
```

## 서비스의 타입

### CluserIP

Pod 그룹의 단일 진입점(Virtual IP) 생성

- selector의 **label이 동일한 파드들의 그룹**으로 묶어 단일 진입점을 생성한다.
- 클러스터 내부에서만 사용가능
- type 생략 시 default 값으로 10.96.0.0/12 범위에서 할당 됨

#### 예제

clusterip-service를 조회해보면 10.100.100.100의 포트 80으로 실행하고 있는 것을 볼 수 있다. 이 아이피는 nginx 3개의 IP를 묶고 있는 것이다.

```console
# Deployment Controller 먼저 생성
$ cat deployment-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webui
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

$ cat clusterip-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  clusterIP: 10.100.100.100
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80


$ kubectl create -f deployment-nginx.yaml
$ kubectl create -f clusterip-nginx.yaml

$ kubectl get service
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
clusterip-service   ClusterIP   10.100.100.100   <none>        80/TCP    11s
kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP   4m22s

# 자세히 조회해보기
$ kubectl describe svc clusterip-service
Name:              clusterip-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=webui
Type:              ClusterIP
IP:                10.100.100.100
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.2:80,10.244.1.3:80,10.244.1.5:80
Session Affinity:  None
Events:            <none>

# 직접 접속해보기 (3개 중 하나로 포워딩)
$ curl 10.100.100.100
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# 세 개의 화면 내용을 바꿔보자
$ kubectl exec webui-6f8645dc9-m9rxg -it /bin/bash
root@webui-6f8645dc9-m9rxg:/# cd /usr/share/nginx/html/
root@webui-6f8645dc9-m9rxg:/usr/share/nginx/html# ls
50x.html  index.html
root@webui-6f8645dc9-m9rxg:/usr/share/nginx/html# echo "webui #1" > index.html
root@webui-6f8645dc9-m9rxg:/usr/share/nginx/html# exit

# 균등하게 할 수 있도록 랜덤하게 동작한다.
$ curl 10.100.100.100
webui #2
$ curl 10.100.100.100
webui #1

# replica 개수가 늘어나도 자동으로 할당이 되어 Virtual IP로 묶인다.
$ kubectl scale deployment webui --replicas=5
deployment.apps/webui scaled
controlplane $ kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
webui-6f8645dc9-d52mq   1/1     Running   0          8s    10.244.1.8   node01   <none>           <none>
webui-6f8645dc9-fs9gv   1/1     Running   0          8s    10.244.1.7   node01   <none>           <none>
webui-6f8645dc9-hwn8s   1/1     Running   0          12m   10.244.1.2   node01   <none>           <none>
webui-6f8645dc9-m9rxg   1/1     Running   0          12m   10.244.1.3   node01   <none>           <none>
webui-6f8645dc9-nrt6k   1/1     Running   0          12m   10.244.1.5   node01   <none>           <none>
```

### NodePort

Cluster IP가 생성된 후 모든 Worker Node에 외부에서 접속가능한

- 모든 노드를 대상으로 외부 접속 가능한 포트를 예약
- Default NodePort 범위 : 30000- 32767
- ClusterIP를 생성 후 NodePort를 예약
- ClusterIP는 내부에서만 사용할 수 있는데 NodePort를 지정해주면 외부에서 접속할 수 있게 된다.

#### 예제

```console
$ cat nodeport-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  clusterIP: 10.100.100.200
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30200 # 지정 안 하면 Default

# 이전에 실습한 내용 지우기
$ kubectl delete service --all
service "clusterip-service" deleted
service "kubernetes" deleted

$ kubectl create -f nodeport-nginx.yaml

$ kubectl get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP        66s
nodeport-service   NodePort    10.100.100.200   <none>        80:30200/TCP   7s


$ ssh node01
Warning: Permanently added 'node01,10.0.0.30' (ECDSA) to the list of known hosts.
node01 $ netstat -napt|grep 30200
tcp        0      0 0.0.0.0:30200           0.0.0.0:*               LISTEN      2222/kube-proxy



# 아무 노드에 접속하면 LB 처럼 랜덤하게 트래픽을 할당한다.
$ curl node01:30200
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
controlplane $ curl node02:30200
webui #1
```

### LoadBalancer

- Public 클라우드(AWS, Azure, GCP 등)에서 운영가능
- LoadBalancer를 자동으로 구성 요청
- NodePort를 예약 후 해당 nodeport로 외부 접근을 허용

#### 예제

```console
$ cat loadbalancer-nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

$ kubectl delete svc --all
$ kubectl create -f loadbalancer-nginx.yaml

# 만약 퍼블릭 클라우드였다면 LB 장비의 address가 EXTERNAL-IP에 있었을 것.
$ kubectl get svc
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes             ClusterIP      10.96.0.1       <none>        443/TCP        39s
loadbalancer-service   LoadBalancer   10.108.174.29   <pending>     80:31227/TCP   5s
```

### ExternalName

클러스터 내부에서 External의 도메인을 설정

```console
$ cat external-name.yaml
apiVersion: v1
kind: Service
metadata:
  name: externalname-svc
spec:
  type: ExternalName
  externalName: google.com

$ kubectl create -f external-name.yaml

$ kubectl get svc
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
externalname-svc   ExternalName   <none>       google.com    <none>    8s

# Name으로 호출하기
$ kubectl run testpod -it --image=centos:7
$ kubectl exec testpod -it /bin/bash

# kubernetes가 내부적으로 사용하는 도메인 뒤에 붙여주기
# google로 포워딩 됨
[root@testpod /]# curl externalname-svc.default.svc.cluster.local
<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 404 (Not Found)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}html{background:#fff;color:#222;padding:15px}body{margin:7% auto 0;max-width:390px;min-height:180px;padding:30px 0 15px}* > body{background:url(//www.google.com/images/errors/robot.png) 100% 5px no-repeat;padding-right:205px}p{margin:11px 0 22px;overflow:hidden}ins{color:#777;text-decoration:none}a img{border:0}@media screen and (max-width:772px){body{background:none;margin-top:0;max-width:none;padding-right:0}}#logo{background:url(//www.google.com/images/branding/googlelogo/1x/googlelogo_color_150x54dp.png) no-repeat;margin-left:-5px}@media only screen and (min-resolution:192dpi){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat 0% 0%/100% 100%;-moz-border-image:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) 0}}@media only screen and (-webkit-min-device-pixel-ratio:2){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat;-webkit-background-size:100% 100%}}#logo{display:inline-block;height:54px;width:150px}
  </style>
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
  <p><b>404.</b> <ins>That’s an error.</ins>
  <p>The requested URL <code>/</code> was not found on this server.  <ins>That’s all we know.</ins>
```

## Kubernetes Headless Service

ClusterIP가 없는 서비스를 Headless 서비스라고 한다. 보통 단일 진입점이 필요 없을 때 사용한다. (만들긴 하는데 IP address가 없음) service와 연결된 Pod의 endpoint로 DNS 레코드가 CoreDNS에 생성됨. 이렇게 되면 Pod의 Endpoint를 DNS resolving service로 요청이 가능하다.

요컨대 Pod들의 endpoint에 DNS resolving service를 지원해주는 것이 목적이다.

- Pod의 DNS 주소 : pod-ip-addr.namespace.pod.cluster.local

### 예제

```console
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  type: ClusterIP
  clusterIP: None # Headless service는 cluster ip를 none으로 설정하면 된다.
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

$ kubectl create -f headless-nginx.yaml
$ kubectl get svc
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
externalname-svc   ExternalName   <none>       google.com    <none>    19m
headless-service   ClusterIP      None         <none>        80/TCP    16s

# 묶여있는 걸 볼 수 있음
$ kubectl describe svc headless-service
Name:              headless-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=webui
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.2:80,10.244.1.3:80,10.244.1.5:80 + 2 more...
Session Affinity:  None
Events:            <none>


# pod 하나 만들기
$ kubectl run testpod2 -it --image=centos:7 /bin/bash
[root@testpod2 /]# cat /etc/resolv.conf
nameserver 10.96.0.10 # -> core DNS의 IP Address
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5

# DNS 서비스 요청해보기
[root@testpod2 /]# curl 10-244-1-5.default.pod.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Pod간 접속을 할 때 사용할 수 있고 Pod의 이름이 자주 바뀌는 서비스 보다는 StatefulSet과 같이 pod이름이 보존되는 경우에 쓰면 좋다.

## Kube-Porxy

실제 Kubernetes Service를 구현시켜주는 backend 서비스

- Endpoint 연결을 위한 iptables를 구성한다.
- 서비스 type을 nodeport로 운영하면 이게 생기고 이 nodeport를 listen하고 있어 client 사용자가 외부에서 들어올 수 있도록 잡고 있다. 그리고 이 rule을 통해 node 중 하나에 연결할 수 있도록 한다.

```console
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS
.
.
.
# 각 노드에서 kube-proxy 동작 중
kube-system   kube-proxy-n9hln                           1/1     Running            0          12m
kube-system   kube-proxy-w5jrq                           1/1     Running            0          12m
kube-system   kube-scheduler-controlplane                1/1     Running            0          12m

# 이 kube-proxy는 각 노드에서 iptables rule을 만들어 다른 노드와 연결할 수 있도록 한다.

# 확인해보기
node01$ iptables -t nat -S | grep 80
-A KUBE-MARK-DROP -j MARK --set-xmark 0x8000/0x8000
```

### 동작방식

1. userspace

- 클라이언트의 서비스 요청을 iptables를 거쳐 kube-proxy가 받아서 연결
- kubernetes 초기 버전에 잠깐 사용

2. **iptables**

- default kubernetes network mode
- kube-porxy는 service API 요청 시 iptables rule이 생성
- 클라이언트 연결은 kube-proxy가 받아서 iptables 룰을 통해 연결

4. IPVS

- 리눅스 커널이 지원하는 L4 로드밸런싱 기술을 이용
- 별도의 ipvs 지원 모듈을 설정한 후 적용가능
- 지원 알고리즘 : rr, lc, dh, sh, sed, nc
