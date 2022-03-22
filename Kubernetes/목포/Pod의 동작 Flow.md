# Pod의 동작 flow

```console
# 아래 명령어를 실행했을 때의 pod의 동작 flow를 알아보자.
$ kubectl run webserver --image=nginx:1.14
```

1. API가 해당 명령이 맞는 문법인지 확인한다.
2. nodee들의 정보가 들어있는 etcd의 정보를 스케줄러에게 보낸다.
3. 스케줄러는 이 웹서버 pod를 어디에 올리면 좋을지 탐색한다.  
   ———————— Pending —————————
4. 워커에 이 pod가 배치되면 Running 상태로 바뀐다.
5. Succeded / Failed

#### Watch Command를 통해 확인해보기

```console
$ kubectl get pods -o wide --watch

$ kubectl create -f pod-nginx.yaml

NAME        READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
webserver   0/1     Pending   0          0s    <none>   <none>   <none>           <none>
webserver   0/1     Pending   0          0s    <none>   node01   <none>           <none>
webserver   0/1     ContainerCreating   0          0s    <none>   node01   <none>           <none>
webserver   1/1     Running             0          4s    10.244.1.4   node01   <none>           <none>


$ kubectl delete pod webserver

webserver   1/1     Terminating         0          36s   10.244.1.4   node01   <none>           <none>
webserver   0/1     Terminating         0          38s   10.244.1.4   node01   <none>           <none>
webserver   0/1     Terminating         0          39s   10.244.1.4   node01   <none>           <none>
webserver   0/1     Terminating         0          39s   10.244.1.4   node01   <none>           <none>

```

watch 커맨드를 통해 pod가 어떻게 생성되고 삭제되는지 확인할 수 있다.

## 정리

### 동작 중인 파드 정보

```console
$ kubectl get pods
$ kubectl get pods -o wide
$ kubectl describe pod webserver
```

### 동작 중인 파드 수정

```console
$ kubectl edit pod webserver
```

### 동작 중인 파드 삭제

```console
$ kubectl delete pod webserver
$ kubectl delete pod -all
```

### 전체 네임스페이스에서 동작 중인 파드 확인하기

```console
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-hmdhl                   1/1     Running   0          7m48s
kube-system   coredns-66bff467f8-wrtmr                   1/1     Running   0          7m47s
kube-system   etcd-controlplane                          1/1     Running   0          7m57s
kube-system   katacoda-cloud-provider-6c68b8fbc5-g2d84   1/1     Running   4          7m45s
kube-system   kube-apiserver-controlplane                1/1     Running   0          7m57s
kube-system   kube-controller-manager-controlplane       1/1     Running   0          7m57s
kube-system   kube-flannel-ds-amd64-8jmgv                1/1     Running   0          7m34s
kube-system   kube-flannel-ds-amd64-cff66                1/1     Running   0          7m49s
kube-system   kube-keepalived-vip-n748s                  1/1     Running   0          7m3s
kube-system   kube-proxy-c578z                           1/1     Running   0          7m34s
kube-system   kube-proxy-d8gzr                           1/1     Running   0          7m48s
kube-system   kube-scheduler-controlplane                1/1     Running   0          7m57s
```
