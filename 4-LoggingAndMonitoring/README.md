# Kubernetes for Kode Kloud - Logging & Monitoring

## 1. Logging & Monitoring
> 도커의 로그를 모니터링 할 수 있으며, 여러 컨테이너가 존재하는 경우 파드 이름과 컨테이너 이름을 나열함으로써 로그 모니터링이 가능합니다

```bash
bash> kubectl logs -f <pod-name> <container-name>
bash> git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
bash> cd kubernetes-metrics-server ; kubectl create -f .

bash> kubectl get pods -n kube-system --watch
NAME                                   READY   STATUS              RESTARTS   AGE
...
metrics-server-774b56d589-8lbff        0/1     ContainerCreating   0          15s

bash> kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
controlplane   122m         6%     1032Mi          54%
node01         1997m        99%    543Mi           14%

bash> kubectl top pod
NAME       CPU(cores)   MEMORY(bytes)
elephant   12m          18Mi
lion       893m         1Mi
rabbit     966m         1Mi
```
