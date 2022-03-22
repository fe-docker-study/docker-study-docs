# Kubernetes Pod-LivenessProbe를 이용한 Self-healing Pod 만들기

## Self-healing?

![54B2E3E8-E3A7-45AF-9363-3B4FDFB6265C](https://user-images.githubusercontent.com/31172248/159274355-f2bb1da7-25cf-4fd4-8c4c-2e1c266574dd.png)

[쿠버네티스 홈페이지]("kubernetes.io")에 가보면 쿠버네티스가 지원하는 기능을 볼 수 있는데, 바로 컨테이너가 죽었거나 내려갔을 때 재배치해주거나, 리스케줄해주는 것을 말한다. 쿠버네티스의 핵심적인 기능이라고 볼 수 있다. Self-healing 안에 포함된 것이 Liveness Probe이다.

## Liveness Probe

컨테이너를 진단하여 이 컨테이너가 건강하다면 계속 진행하고 그렇지 않다면 재시작한다. 그렇다면 Liveness Probe가 포함된 Pod와 그렇지 않은 Pod의 Pod Template을 통해 알아보자.

```console
# Pod-definiation
apiVersion: v1
kind: Pod
metadata:
	name: nginx-pod
spec:
	containers:
	- name: nginx-container
	  image: nginx:1.14


# livenessProbe definition (Spec에 정의)
apiVersion: v1
kind: Pod
metadata:
	name: nginx-pod
spec:
	containers:
	- name: nginx-container
	  image: nginx:1.14
	  livenessProbe:
		httpGet:
			path: / # (root page)
			prot: 80
```

livenessProbe 중 httpGet으로 (WebService)로 건강하게 동작 중인지 확인하겠다는 의미이다. 즉, ipaddress:80/의 경로로 주기적 접속을 했을 때 응답이 잘 나온다면(200) 건강한 컨테이너라고 판단하는 것이다.

하지만, 서비스 중엔 웹서비스만 있는 것이 아니기 때문에 서비스마다 이렇게 검사를 하는 방법이 다르다. 보통 3가지 형태로 나눠 지원해주고 있다. livenessProbe의 매커니즘을 알아보자.

### LivenessProbe 매커니즘

- httpGet probe
  - 지정한 IP 주소, port, path에 HTTP GET 요청을 보내, 해당 컨테이너가 응답을하는지 확인한다. 반환코드가 200이 아닌 값이 나오면 오류, 컨테이너를 다시 시작한다.
  - 보통 연속 3번 실패하면 건강하지 않은 컨테이너로 판단한다.
    - 건강하지 않은 컨테이너를 지우고, Docker Hub에서 다시 내려받아 Restart한다.

```console
livenessProbe:
	httpGet:
	  path: /
	  prot: 80
```

- tcpSocket probe
  - 지정된 포트에 TCP 연결을 시도. 연결되지 않으면 컨테이너를 다시 시작한다.
  - 22번 포트로 클라이언트 커넥션을 받아주는 서비스일 것이다. 22번 접속이 성공하면 건강한 컨테이너라 판단하고, 이 역시 3번 실패 시 해당 pod를 지우고 새 pod를 받아다가 restart한다.

```console
livenessProbe:
	tcpSocket:
	  prot: 22
```

- exec probe
  - exec 명령을 전달하고 명령의 종료코드가 0이 아니면 컨테이너를 다시 시작한다.
  - ex) 이 컨테이너에서 유효한 데이터를 확인할 수 있는 명령어를 적어준다. (ls ps, ls /data/file 등..)

```console
livenessProbe:
	exec:
	command:
	  - ls
	  - /data/file
```

여기서 restart한다는 것은 컨테이너를 지웠다가 새로 받아온다는 것이다. pod는 그대로 있기 때문에 pod에 할당된 ip는 변하지 않는다.
