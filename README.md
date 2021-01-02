# KODE KLOUDE Labs

## YAML of Pods, ReplicaSets, Deployments
| Type | Pod | Service | RelicaSet | Deployment |
|---|---|---|---|---|
| YAML | `apiVersion: v1` | `apiVersion: v1` | `apiVersion: apps/v1` | `apiVersion: apps/v1` |
|      | `kind: Pod` | `kind: Service` | `kind: ReplicaSet` | `kind: Deployment` |
|      | `` | `` | `` | `` |
|      | `` | `` | `` | `` |
|      | `` | `` | `` | `` |
|      | `` | `` | `` | `` |


## Practice Tests - PODs
* yaml 대신 명령어를 통해서 컨테이너 기동하는 방법
```bash
bash> kubectl run nginx --image nginx

bash> cat pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
    - name: nginx
      image: nginx

bash> kubectl create -f pod-definition.yml

bash> kubectl run redis 
```

## Practice Tests - ReplicaSets
* 아래와 같이 selector:tier 의 값과 metadata:labels:tier 가 같아야만 합니다
```bash
bash> cat /root/replicaset-definition-2.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: busybox-app
  template:
    metadata:
      labels:
        tier: busybox-app
    spec:
      containers:
      - name: nginx
        image: nginx
```


* 레플리카셋 예제에서 busybox 데몬 형태로 떠 있도록 하려면
  - /bin/sh 명령을 통해 sleep 을 아래와 같이 추가합니다
```bash
bash> cat /root/replicaset-definition-3.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-3
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: busybox-app
  template:
    metadata:
      labels:
        tier: busybox-app
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "echo Hello Kubernetes; sleep 3600"]
```

## Practice Tests - Deployments
* 간단한 디플로이먼트 생성
```bash
bash> cat deployment-definition-1.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
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
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
```

* 아래와 같은 디플로이먼트 수정
  - Name: httpd-frontend; Replicas: 3; Image: httpd:2.4-alpine
  - deployments 의 이름은 metadata 를 통해 생성되며, container 의 이름은 spec 의 name 을 통해 생성됩니다
```bash
bash> cat deployment-definition-3.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-definition-3
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

```
