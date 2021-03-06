<aside>
๐ ์๋น์ค = ์ฟ ๋ฒ๋คํฐ์ค ๋คํธ์ํฌ

</aside>

## 1. Kubernetes Service ๊ฐ๋

- ๋์ผํ ์๋น์ค๋ฅผ ์ ๊ณตํ๋ **Pod ๊ทธ๋ฃน์ ๋จ์ผ ์ง์์ **์ ์ ๊ณต
    
    ![image](https://user-images.githubusercontent.com/47748246/162397131-e960bd02-0e33-4a9a-b933-a68c226191b6.png)
    

### Service definition

![image](https://user-images.githubusercontent.com/47748246/162397167-945682a9-f88a-4358-af82-b7a5949f38bc.png)
## 2. Kubernetes Service ํ์

- 4๊ฐ์ง Type ์ง์
    - ClusterIP (default)
        - Pod ๊ทธ๋ฃน์ **๋จ์ผ ์ง์์ (Virtual IP(=loadbalancer ip))** ์์ฑ
    - NodePort
        - ๊ธฐ๋ณธ์ ์ผ๋ก ClusterIP๊ฐ ์์ฑ๋จ
        - ์ถ๊ฐ๋ก, ๋ชจ๋  Worker Node์ ์ธ๋ถ์์ ์ ์ ๊ฐ๋ฅํ ํฌํธ๊ฐ open
    - LoadBalancer
        - NodePort + LB ์ฅ๋น
            - ์ค์  LB ์ฅ๋น์ ip address์ NodePort๋ฅผ ์ฐ๊ฒฐ์์ผ์ค
        - ํด๋ผ์ฐ๋ ์ธํ๋ผ์คํธ๋ฝ์ฒ(AWS, Azure, GCP ๋ฑ)๋ ์คํ ์คํ ํด๋ผ์ฐ๋์ ์ ์ฉ
        - LoadBalancer๋ฅผ ์๋์ผ๋ก ํ๋ก ๋น์ ํ๋ ๊ธฐ๋ฅ ์ง์
    - ExternalName
        - ํด๋ฌ์คํฐ ์์์ ์ธ๋ถ์ ์ ์ ์ ์ฌ์ฉํ  **๋๋ฉ์ธ์ ๋ฑ๋ก**ํด์ ์ฌ์ฉ
        - ํด๋ฌ์คํฐ ๋๋ฉ์ธ์ด ์ค์  ์ธ๋ถ ๋๋ฉ์ธ์ผ๋ก ์นํ๋์ด ๋์

### ClusterIP

- ์ฟ ๋ฒ๋คํฐ์ค์๊ฒ deploy๋ฅผ ํตํด์ ์์ฒญ
    - โ์น ์๋ฒ๋ฅผ webui๋ผ๋ ์ด๋ฆ์ผ๋ก 3๊ฐ ์ด์ํด์คโ
    - ์ด๋ ์์ฑ๋ webui ํ๋๋ค์ **๊ฐ์ ์๋ก ๋ค๋ฅธ  ip๋ฅผ ๊ฐ์ง**
- ์ด์ฒ๋ผ selector์ label์ด ๋์ผํ ํ๋๋ค์ ๊ทธ๋ฃน์ผ๋ก ๋ฌถ์ด์ฃผ๊ธฐ ์ํด service API๊ฐ ํ์ํ๋ค.
- ์ฆ, Service API๋ฅผ ํตํด **๋จ์ผ ์ง์์  (Virtual IP)๋ฅผ ์์ฑ**ํ์ฌ ๋์ผํ ํ๋๋ค(label๋ก ๋์ผํ์ง ์ฌ๋ถ๋ฅผ ํ์ธ)์ ๋ก๋๋ฐธ๋ฐ์ฑํ๋ค.
- type ์๋ต ์ default ๊ฐ์ผ๋ก 10.06.0.0/12 ๋ฒ์์์ ํ ๋น๋จ

์์ 

![image](https://user-images.githubusercontent.com/47748246/162397250-8e8a131d-ab03-4116-bff1-ea89fe748b19.png)

```json
$ kubectl create -f clusterip-nginx.yaml
$ kubectl get svc

$ curl 10.100.100.100
$ kubectl describe svc clusterip-service
$ kubectl delete svc clusterip-service
```

### NodePort

- ๋ชจ๋  ๋ธ๋๋ฅผ ๋์์ผ๋ก **์ธ๋ถ ์ ์ ๊ฐ๋ฅํ ํฌํธ๋ฅผ ์์ฝ**
- Default NodePort ๋ฒ์ : 30000 - 32767
- ClusterIP๋ฅผ ์์ฑํ ํ NodePort๋ฅผ ์์ฝ

์์ 

![image](https://user-images.githubusercontent.com/47748246/162397311-399ae37d-3d73-4826-96a2-4316320c9021.png)

### LoadBalancer

- LoadBalancer๋ Public ํด๋ผ์ฐ๋ (AWS, Azure, GCP ๋ฑ)์์ ์ด์ ๊ฐ๋ฅ
- LoadBalancer๋ฅผ ์๋์ผ๋ก ๊ตฌ์ฑ ์์ฒญ
- NodePort๋ฅผ ์์ฝ ํ ํด๋น nodeport๋ก ์ธ๋ถ ์ ๊ทผ์ ํ์ฉ

![image](https://user-images.githubusercontent.com/47748246/162397355-69f34232-596e-4750-a42d-6c3c6a1ec567.png)

![image](https://user-images.githubusercontent.com/47748246/162397398-029f159f-cadf-40d1-8088-c29fdd994bf9.png)

### ExternalName

- ํด๋ฌ์คํฐ ๋ด๋ถ์์ External(์ธ๋ถ)์ ๋๋ฉ์ธ์ ์ค์ 

![image](https://user-images.githubusercontent.com/47748246/162397438-498f3530-3b7d-48d7-8e41-9cbe826708ac.png)

![image](https://user-images.githubusercontent.com/47748246/162397479-8e7c1925-e49d-4ff1-9d28-967fb09a19ca.png)

## 3. Headless Service

- ClusterIP๊ฐ ์๋ ์๋น์ค. ๋จ์ผ ์ง์์ ์ด ํ์ ์์ ๋ ์ฌ์ฉ
    - endpoint๋ฅผ ๋ฌถ์ด์ฃผ๊ธด ํ์ง๋ง ๊ทธ์ ๋ํ ip address๋ ์์
- Service์ ์ฐ๊ฒฐ๋ Pod์ endpoint์ ๋ํ DNS ๋ ์ฝ๋๊ฐ core DNS์ ์์ฑ๋จ
    
    โ pod์ ๋ํ endpoint๋ฅผ DNS resolving ์๋น์ค๋ก ์์ฒญํ  ์ ์์
    
- Pod์ DNS ์ฃผ์ : pod-ip-addr.namespace.pod.cluster.local

![image](https://user-images.githubusercontent.com/47748246/162397524-660dcdcf-3c26-47f9-a600-41ebc1787e20.png)

## 4. kube-proxy

- Kubernetes Service์ backend ๊ตฌํ
- endpoint ์ฐ๊ฒฐ์ ์ํ iptables ๊ตฌ์ฑ
- nodePort๋ก์ ์ ๊ทผ๊ณผ pod ์ฐ๊ฒฐ์ ๊ตฌํ (iptables ๊ตฌ์ฑ)

### Kube-proxy mode

- userspace
    - ํด๋ผ์ด์ธํธ์ ์๋น์ค ์์ฒญ์ iptables๋ฅผ ๊ฑฐ์ณ kube-proxy๊ฐ ๋ฐ์์ ์ฐ๊ฒฐ
    - kubernetes ์ด๊ธฐ ๋ฒ์ ์ ์ ๊น ์ฌ์ฉ
- iptables
    - defualt kubernetes network mode
    - kube-proxy๋ service API ์์ฒญ ์ iptables rule์ด ์์ฑ
    - ํด๋ผ์ด์ธํธ ์ฐ๊ฒฐ์ kube-proxy๊ฐ ๋ฐ์์ iptables ๋ฃฐ์ ํตํด ์ฐ๊ฒฐ
- IPVS
    - ๋ฆฌ๋์ค ์ปค๋์ด ์ง์ํ๋ L4 ๋ก๋๋ฐธ๋ฐ์ฑ ๊ธฐ์ ์ ์ด์ฉ
    - ๋ณ๋์ ipvs ์ง์ ๋ชจ๋์ ์ค์ ํ ํ ์ ์ฉ ๊ฐ๋ฅ
    - ์ง์ ์๊ณ ๋ฆฌ์ฆ : rr(round-robin), lc(least connection), dh (destination hashing), sh(source hashing), sed(shortest expected delay), nc
