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


## Practice Tests - Deployments
* 간단한 디플로이먼트 생성
  - deployment 기동을 위한 yaml 파일 생성
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


## YAML of Pods, ReplicaSets, Deployments
* Pod
  - 파드 기동을 위한 yaml 파일 생성
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

* Service
  - 서비스 기동을 위한 yaml 파일 생성
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
  - minikube 이용시에는 external-ip 가 없기 때문에, service url 확인이 필요합니다
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

* ReplicaSet
  - replicaset 기동을 위한 yaml 파일 생성
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
  - replicaset 의 경우 replicaset -> service -> pod 까지 생성됩니다
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

* Deployment
  - deployment 기동을 위한 yaml 파일을 생성합니다
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
        command: ["/bin/sh", "-c", "echo Hello Kubernetes! && sleep 3600"]
```
  - deployment 의 경우 deployment -> replicaset -> service -> pod 까지 생성됩니다
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

