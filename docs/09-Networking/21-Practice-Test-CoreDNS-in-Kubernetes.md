# Practice Test CoreDNS in Kubernetes

- Take me to [Practice Test](https://kodekloud.com/topic/practice-test-coredns-in-kubernetes/)

## Solution

### 1. Check the Solution

```
CoreDNS
```

`kubectl get pods -n kube-system` and look for the DNS pods.

```bash
kubectl get po -A -o wide | grep dns
```

```
kube-system    coredns-6678bcd974-btxxb               1/1     Running
```

```bash
kubectl get svc -A
```

```
NAMESPACE     NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
kube-system   kube-dns       ClusterIP   172.20.0.10      <none>        53/UDP,53/TCP,9153/TCP   9m45s
```

```bash
kubectl get deployment -A | grep coredns
```

```
kube-system   coredns   2/2     2            2           11m
```

Where is the configuration file located for configuring the CoreDNS service?

```bash
controlplane ~ ✖ kubectl get deployment coredns -n kube-system  -oyaml | grep conf
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"k8s-app":"kube-dns"},"name":"coredns","namespace":"kube-system"},"spec":{"replicas":2,"selector":{"matchLabels":{"k8s-app":"kube-dns"}},"strategy":{"rollingUpdate":{"maxUnavailable":1},"type":"RollingUpdate"},"template":{"metadata":{"labels":{"k8s-app":"kube-dns"}},"spec":{"affinity":{"podAntiAffinity":{"preferredDuringSchedulingIgnoredDuringExecution":[{"podAffinityTerm":{"labelSelector":{"matchExpressions":[{"key":"k8s-app","operator":"In","values":["kube-dns"]}]},"topologyKey":"kubernetes.io/hostname"},"weight":100}]}},"containers":[{"args":["-conf","/etc/coredns/Corefile"],
```

### 2. Check the Solution

```
2
```

### 3. Check the Solution

```
10.96.0.10
```

### 4. Check the Solution

```
/etc/coredns/Corefile

OR

kubectl -n kube-system describe deployments.apps coredns | grep -A2 Args | grep Corefile
```

### 5. Check the Solution

```
Configured as a ConfigMapObject
```

### 6. Check the Solution

```
CoreDNS
```

### 7. Check the Solution

```
coredns
```

### 8. Check the Solution

What is the root domain/zone configured for this kubernetes cluster?

```bash
kubectl describe configmap coredns -n kube-system
```

```
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}

BinaryData
====

Events:  <none>
```

```
cluster.local
```

### 9. Check the Solution

```
Ok
```

### 10. Check the Solution

```
web-service
```

### 11. Check the Solution

```
web-serivce.default.pod
```

### 12. Check the Solution

```
web-service.payroll
```

### 13. Check the Solution

```
web-service.payroll.svc.cluster
```

### 14. Check the Solution

```
kubectl edit deploy webapp

Search for DB_Host and Change the DB_Host from mysql to mysql.payroll

spec:
  containers:
  - env:
    - name: DB_Host
      value: mysql.payroll
```

### 15. Check the Solution

```
kubectl exec -it hr -- nslookup mysql.payroll > /root/nslookup.out
```

```bash
kubectl exec hr -- nslookup mysql.payroll > /root/CKA/nslookup.out
```

```bash
controlplane /etc ➜  cat /root/CKA/nslookup.out
```

```
Server:         172.20.0.10
Address:        172.20.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 172.20.126.129
```
