# Kubernetes for Kode Kloud - Security
> 

## 쿠버네티스 서비스계정 생성
```bash
bash> kubectl create serviceaccount sa1
bash> kubectl list serviceaccount
bash> curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```
* 패스워드 혹은 토큰 등을 통한 인증
  - 패스워드 파일 대신, 볼륨 마운트를 통한 인증을 검토하라
```yaml
# cat /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```
* kube-apiserver 기동 시에 옵션을 변경합니다
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
      <content-hidden>
    - --basic-auth-file=/tmp/users/user-details.csv
```
* 롤과 롤바인딩을 생성합니다
```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
 
---
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

## TLS (Transport Layer Security) 인증 
> 암호화된 정보와 키를 같이 보내어 해독하는 방식을 대칭형 암호화(Symmetric Encryption)이라고 합니다. 평문 보다는 높은 보안이지만, 패킷캡쳐를 통해 해독이 가능합니다. 하여 비대칭 암호화 즉, Private Key 와 Public Lock 인 경우를 말하는데, Private Key 는 암호화만 가능하며, Public Key 는 해독만 가능합니다.


## [에단의 TLS](https://www.youtube.com/watch?v=EPcQqkqqouk)
* tracerout 명령을 통해 특정 URL 혹은 IP 로 접근하는 과정을 말해주는 도구 (윈도우는 tracert)
  - 나와 직접 서버와 연결되지 않고, 상당히 많은 과정을 거칩니다


## Practice Test View Certificate Details
* API, ETCD 서버의 인증(Certificate)정보 확인
```bash
bash> openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
bash> openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text
bash> openssl x509 -in /etc/kubernetes/pki/ca.crt -text
```
* ETCD 설정 파일 확인을 통해 문제점 파악하기
  - cert-file 위치 확인 /etc/kubernetes/pki/etcd/server.crt
```bash
bash> vi /etc/kubernetes/manifests/etcd.yaml
```


## 신규 관리자 인증서 추가
* openssl 명령을 통해 인증서를 생성합니다
```bash
bash> openssl genrsa -out suhyuk.key 2048
bash> openssl req -new -key suhyuk.key -subj "/CN=suhyuk" -out suhyuk.csr
```
* 생성된 인증파일을 통해 인증을 요청합니다
  - "cat suhyuk.csr | base64"
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: suhyuk
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGN
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```
* 추가된 계정을 인증합니다
```bash
bash> kubectl get csr
... Pending
bash> kubectl certificate approve suhyuk
bash> kubectl certificate deny agent-smith
bash> kubectl delete csr agent-smith
bash> kubectl get csr suhyuk -o yaml
```
* 인증작업을 Scheduler 를 통해 수행되며, 백엔드에 ControllerManager 에 의해 운영됩니다
```bash
bash> cat /etc/kubernetes/manifests/kube-controller-manager.yaml
spec:
  containers:
  - command:
    - kube-controller-manager
    - --address=127.0.0.1
    - --cluster-signinig-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signinig-key-file=/etc/kubernetes/pki/ca.key
    ...
```


## KubeConfig 실습
```bash
bash> curl https://me-kube-playground:6443/api/v1/pods \
	--key admin.key \
	--cert admin.crt \
	--cacert ca.crt

bash> kubectl get pods \
	--server me-kube-playground:6443 \
  --client-key admin.key \
	--client-certificate admin.crt \
  --certificate-authority ca.crt
```
* 3개의 섹션으로 구성된 Config 파일을 생성합니다
  - 하나의 클러스터에 대한 구성
```yaml
apiVersion: v1
kind: Config
clusters:
- name: suhyuk-kube-playground
	cluster:
		certificate-authority: ca.crt
		server: https://suhyuk-kube-playground:6443
contexts:
- name: suhyuk-kube-admin@suhyuk-kube-playground
	context:
		cluster: suhyuk-kube-playground
		user: suhyuk-kube-admin
users:
- name: suhyuk-kube-admin
  user:
		client-certificate: admin.crt
		client-key: admin.key
```
  - 여러개의 클러스터에 대한 구성을 하며 current-context 를 통해 default context 를 설정합니다
```yaml
apiVersion: v1
current-context: psyoblade@ncsoft
kind: Config
clusters:
- name: suhyuk-kube-playground
- name: dev-kube-playground
- name: prod-kube-playground
- name: gcp-kube-playground
...
contexts:
- name: suhyuk-kube-admin@suhyuk-kube-playground
- name: psyoblade@ncsoft
...
users:
- name: suhyuk-kube-admin
...
```
* Config 보기 및 변경
```bash
bash> kubectl config view
bash> kubectl config view --kubeconfig=suhyuk-kube-playground
bash> kubectl config use-context prod-user@production
bash> kubectl config -h
```
```yaml
apiVersion: v1
kind: Config
clusters:
- name: production
	cluster:
		certificate-authority: /etc/kubernetes/pki/ca.crt
		certificate-authority-data: `cat /etc/kubernetes/pki/ca.crt | base64`
```