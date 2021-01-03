# KODE KLOUDE Labs


## Practice Tests - Pods
* yaml 대신 명령어를 통해서 컨테이너 기동하는 방법
  - 파드 기동을 위한 yaml 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx-app
spec:
  containers:
    - name: nginx-container
      image: nginx
```
  - yaml 이용하거나 혹은 직접 기동하는 방법
```bash
bash> kubectl create -f pod-nginx.yml
bash> kubectl run nginx --image nginx
```

## Practice Tests - ReplicaSets
* 아래와 같이 selector:tier 의 값과 metadata:labels:tier 가 같아야만 합니다
  - replicaset 기동을 위한 yaml 파일 생성
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-busybox
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "echo Hello Kubernetes; sleep 3600"]
```
* replicas 변경 - 파일을 변경하거나, scale 명령을 통해 변경이 가능합니다
```bash
bash> kubectl replace -f replicaset-busybox-modified.yaml
bash> kubectl scale --replicas=3 -f replicaset-busybox.yaml
bash> kubectl scale --replicas=4 replicaset replicaset-busybox
```
* apply 는 replace 와 다르게 in-place 변경을 수행하므로 apply 한 객체에 대해서 적용하는 것이 적절하다
> The difference is that replace first deletes the resources, then creates it from the file you give it; whereas apply attempts to directly update, in the current live resource, only the attributes you give it in the file. See In-place updates and disruptive updates.



## Practice Tests - Deployments
* 간단한 디플로이먼트 생성 - deployment 기동을 위한 yaml 파일 생성
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-busybox  # 디플로이먼트를 통해 생성된 모든 객체에 붙게되는 이름
  labels:
    app: busybox-app-deploy # 현재 디플로이먼트의 이름
    type: backend           # 현재 디플로이먼트의 유형
spec:
  replicas: 2               # 복제 수
  selector:
    matchLabels:
      name: busybox-pod     # 파드 리플리케이션을 위한 라벨
      type: backend         # name, type 이 모두 일치하는 파드에 대한 리플리케이션
  template:
    metadata:
      labels:
        name: busybox-pod   # 현재 파드의 이름과, 유형
        type: backend
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "echo Hello Kubernetes; sleep 3600"]
```
* 파드, 서비스 부터 디플로이먼트까지 수동으로 배포합니다
```bash
bash> kubectl run nginx-pod --image=nginx:alpine
bash> kubectl run redis --image=redis:alpine --labels=tier=db
bash> kubectl expose pod redis --name redis-service --port 6379 --target-port 6379
bash> kubectl create deployment webapp --image=kodekloud/webapp-color
bash> kubectl scale deployment --replicas=3 webapp
bash> kubectl run custom-nginx --image=nginx:alpine --port 8080
```
* 임의의 yaml 파일을 생성해서 수정 후 배포합니다
```bash
bash> kubectl create ns dev-ns
bash> kubectl create deployment redis-deploy --image=redis:alpine -n dev-ns
bash> kubectl scale deployment --replicas=2 redis-deploy -n dev-ns --dry-run=client -o yaml > deployment-redis.yaml
bash> vi redis.yaml # replicas 를 2로 수정후
bash> kubectl apply -f deployment-redis.yaml
bash> kubectl get deployment.apps -n dev-ns
```
* 명령행에서 yaml 출력해보고 의도한 설정이 나오면 생성합니다 - 기본 서비스는 ClusterIP Type 이 됩니다
```bash
bash> kubectl run httpd --image=httpd:alpine --port 80 --expose --dry-run=client -o yaml 
bash> kubectl describe service httpd
```

## Practice Test - Namespaces
* 같은 혹은 다른 네임스페이스에 존재하는 서비스 접근
  - 같은 네임스페이스에서는 서비스명으로 접근 ex_ db-service
  - 다른 네임스페이스에서는 FQDN 으로 접근 db-service.dev.svc.cluster.local (Service: 서비스명.네임스페이스.svc.cluster.local)
```bash
bash> kubectl get pods --all-namespaces
bash> kubectl describe service db-service
```





## 2. YAML 예제 - from Pod to Deployment
### 2-1. Pod - 파드 기동을 위한 yaml 파일 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx-app
spec:
  containers:
    - name: nginx-container
      image: nginx
```

### 2-2. Service - 서비스 기동을 위한 yaml 파일 생성
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
spec:
  selector:
    app: nginx-app
  ports:
    - protocol: TCP
      port: 80
```
#### minikube 이용시에는 external-ip 가 없기 때문에, service url 확인이 필요합니다
```bash
bash> minikube service service-nginx --url
😿  service default/service-nginx has no node port
🏃  Starting tunnel for service service-nginx.
|-----------|---------------|-------------|------------------------|
| NAMESPACE |     NAME      | TARGET PORT |          URL           |
|-----------|---------------|-------------|------------------------|
| default   | service-nginx |             | http://127.0.0.1:58412 |
|-----------|---------------|-------------|------------------------|
```

### 2-3. ReplicaSet - replicaset 기동을 위한 yaml 파일 생성
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
```
#### replicaset 의 경우 replicaset -> service -> pod 까지 생성됩니다
```bash
bash> kubectl get all

NAME                         READY   STATUS              RESTARTS   AGE
pod/replicaset-nginx-7hq7m   0/1     ContainerCreating   0          3s
pod/replicaset-nginx-smps5   0/1     ContainerCreating   0          3s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   18d

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/replicaset-nginx   2         2         0       3s
```

### 2-4. Deployment - deployment 기동을 위한 yaml 파일을 생성합니다
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-busybox
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "echo Hello Kubernetes; sleep 3600"]
```
#### deployment 의 경우 deployment -> replicaset -> service -> pod 까지 생성됩니다
```bash
bash> kubectl get all

NAME                                      READY   STATUS              RESTARTS   AGE
pod/deployment-busybox-7c8b5b5688-5bhls   0/1     ContainerCreating   0          4s
pod/deployment-busybox-7c8b5b5688-tgl5t   0/1     ContainerCreating   0          4s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   18d

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-busybox   0/2     2            0           4s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-busybox-7c8b5b5688   2         2         0       4s
```

### 2-5. Namespace 및 DNS 접근
* [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

| Kind | DNS Rule | Example |
|---|---|---|
| Service | 서비스명.네임스페이스.svc.cluster.local | nginx-service.default.svc.cluster.local |
| Pod | 파드주소.네임스페이스.pod.cluster.local | 172-17-0-3.default.pod.cluster.local |

