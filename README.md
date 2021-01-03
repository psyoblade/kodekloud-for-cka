# Kubernetes for Kode Kloud

## 1. YAML 예제 - from Pod to Deployment

### 1-1. Pod - 파드 기동을 위한 yaml 파일 생성
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

### 1-2. Service - 서비스 기동을 위한 yaml 파일 생성
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

### 1-3. ReplicaSet - replicaset 기동을 위한 yaml 파일 생성
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

### 1-4. Deployment - deployment 기동을 위한 yaml 파일을 생성합니다
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

### 1-5. Namespace 및 DNS 접근
* [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

| Kind | DNS Rule | Example |
|---|---|---|
| Service | 서비스명.네임스페이스.svc.cluster.local | nginx-service.default.svc.cluster.local |
| Pod | 파드주소.네임스페이스.pod.cluster.local | 172-17-0-3.default.pod.cluster.local |

* Multiple [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) 사용법
  - 현재 사용 중인 네임스페이스 변경
```bash
bash> kubectl set-context --current --namespace=dev-ns
bash> kubectl set-context --current --namespace=default
```


### 1-6. Kubectl Apply 동작방식
> 로컬 YAML 파일을 K8S 에 적용이후에 [metadata.annotations.kubectl.kubernetes.io/last-applied-configuration](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/) 위치에 어노테이션으로 마지가 적용 설정이 저장되며, 로컬에서 삭제된 정보와 서버에 반영된 정보를 비교하기 위해 과거 정보를 유지하며, apply 시에 서버의 설정에 반영하고, 마지막 설정을 해당 어노테이션에 저장해 둡니다
* 사용 시에 유의해야 할 점은 create, apply, replace 명령을 섞어서 사용하게 되면 이전 정보가 제대로 남지 않게 되어 문제가 될 수 있습니다
* apply 로 생성한 경우에는 계속 apply 를 사용해야하며, replace 는 삭제 후 생성이므로, apply 와 섞어서 사용하지 않아야 합니다


