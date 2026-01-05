# Practice Test CKA Ingress 1

## Ingress 流量转发流程

外部请求  
    ↓  
Ingress Controller (ingress-nginx)    
    ↓  根据 Ingress 规则匹配 path=/pay  
     
Service: pay-service (ClusterIP: 172.20.182.49, port: 8282)  
    ↓  
后端 Pod（运行 pay-service 应用，监听容器端口 8282）    

### 请求完整流程（从外部访问到应用 Pod）

1. 用户发起请求：
   ```bash
   curl http://<节点公网IP>:30080/pay
   ```

        ↓
2. 流量到达 Kubernetes 节点的 30080 端口  
   → 该端口由 NodePort 类型的 Service 监听：  
      Namespace: ingress-nginx  
      Service: ingress-nginx-controller  
      映射关系：80 (Service port) → 30080 (NodePort)  

        ↓
3. kube-proxy 将流量转发给 ingress-nginx-controller Pod  
   （Pod 运行真正的 Nginx 反向代理进程）  

        ↓
4. ingress-nginx Controller 读取所有 Ingress 规则  
   → 找到匹配的规则：  
        Namespace: critical-space  
        Ingress 名称: critical-ingress  
        path: /pay  →  转发到 Service: pay-service:8282  

        ↓
5. Controller 查询 Kubernetes API：  
   "pay-service 在 critical-space 命名空间的 ClusterIP 和端口是什么？"  

   → 得到结果：172.20.182.49:8282  

        ↓
6. Controller 向 172.20.182.49:8282 发起 HTTP 请求  
   （Kubernetes 内部网络可达）  

        ↓
7. Service "pay-service" 接收请求  
   → 根据 selector 找到后端 Pod 列表（如 10.244.1.10:8282）  
   → 负载均衡选择一个 Pod  

        ↓
8. 请求到达目标 Pod 的容器  
   → 容器内应用（如 Flask、Spring Boot）处理 /pay 请求  

        ↓
9. 响应原路返回给用户  

- Take me to [Practice Test](https://kodekloud.com/topic/practice-test-cka-ingress-networking-1/)

## Solution

### 1. Check the Solution

```
Ok
```

### 2. Check the Solution

```
INGRESS-SPACE
```

### 3. Check the Solution

What is the name of the Ingress Controller Deployment?

```bash
kubectl get deploy -A | grep ingress
```

```
ingress-nginx   ingress-nginx-controller   1/1     1            1           3m8s
```

```
NGINX-INGRESS-CONTROLLER
```

### 4. Check the Solution

```
APP-SPACE
```

### 5. Check the Solution

```
3
```

### 6. Check the Solution

Which namespace is the Ingress Resource deployed in?

```
APP-SPACE
```

### 7. Check the Solution

What is the name of the Ingress Resource?

```
INGRESS-WEAR-WATCH
```

### 8. Check the Solution

What is the Host configured on the Ingress Resource?

The host entry defines the domain name that users use to reach the application like www.google.com

```bash
kubectl describe ingress ingress-wear-watch   -n app-space
```

```
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          172.20.113.158
Ingress Class:    <none>
Default backend:  <default>  # 默认后端
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (172.17.0.4:8080)
              /watch   video-service:8080 (172.17.0.6:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    11m (x2 over 11m)  nginx-ingress-controller  Scheduled for sync
```

```bash
kubectl get ingress ingress-wear-watch   -n app-space -oyaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2026-01-02T23:52:34Z"
  generation: 1
  name: ingress-wear-watch
  namespace: app-space
  resourceVersion: "2127"
  uid: 000c5ecc-081e-4857-83fd-d2256e22587b
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /watch
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - ip: 172.20.113.158
```

```
ALL-HOSTS(*)
```

### 9. Check the Solution

```
WEAR-SERVICE
```

### 10. Check the Solution

```
/WATCH
```

### 11. Check the Solution

```
DEFAULT-HTTP-BACKEND
```

### 12. Check the Solution

ingress service是NodePort类型，

```bash
kubectl get svc -A | grep ingress-nginx
```

```
ingress-nginx   ingress-nginx-controller             NodePort    172.20.113.158   <none>        80:30080/TCP,443:32103/TCP   18m
ingress-nginx   ingress-nginx-controller-admission   ClusterIP   172.20.5.28      <none>        443/TCP                      18m
```

1. 命名空间（Namespace）：ingress-nginx
   表示该 Service 所在的 Kubernetes 命名空间是 ingress-nginx。通常 Ingress Controller（如 ingress-nginx）会被部署在这个命名空间中。

2. Service 名称（NAME）：ingress-nginx-controller
   这是 Service 的名称，对应的是 Ingress Controller 的入口服务。它负责接收外部 HTTP/HTTPS 请求，并根据 Ingress 规则将流量转发到集群内的后端服务。

3. 类型（TYPE）：NodePort
   说明这个 Service 的类型是 NodePort。这意味着：
   - Kubernetes 会在每个节点（Node）上开放一个固定的高编号端口（如 30080、32103）。
   - 外部用户可以通过 **访问任意节点的 IP 地址加上该端口（如 http://<节点IP>:30080）** 来访问服务。
   - 这是比 ClusterIP 更对外暴露的方式，但不如 LoadBalancer 方便（后者通常用于云平台自动创建负载均衡器）。

4. 集群 IP（CLUSTER-IP）：172.20.113.158
   这是该 Service 在 Kubernetes 集群内部的虚拟 IP 地址。集群内的 Pod 可以通过这个 IP 访问该 Service。但注意，由于类型是 NodePort，这个 IP 通常不是主要访问方式。

5. 外部 IP（EXTERNAL-IP）：<none>
   表示没有分配外部 IP（比如通过云服务商的 LoadBalancer 分配的公网 IP）。这很常见于本地或裸金属集群，或者未配置 LoadBalancer 类型的服务。

6. 端口映射（PORT(S)）：80:30080/TCP,443:32103/TCP
   表示：
   - Service 内部监听 80 端口（HTTP） 和 443 端口（HTTPS）。
   - 这些端口分别被映射到宿主机（Node）的 30080 和 32103 端口。
   - 因此，你可以通过以下方式从集群外部访问 Ingress：
     - HTTP：http://<任意节点IP>:30080
     - HTTPS：https://<任意节点IP>:32103
   - 注意：这些 NodePort（30080、32103）是由 Kubernetes 自动分配的（除非在 Service YAML 中显式指定），范围通常在 30000–32767。

7. 运行时间（AGE）：18m
   表示这个 Service 已经运行了 18 分钟。

总结
这个 Service 是 ingress-nginx 控制器的入口点，通过 NodePort 模式暴露 HTTP（80 → 30080）和 HTTPS（443 → 32103）端口。你可以在浏览器或 curl 中通过访问任意 Kubernetes 节点的 IP 加上对应端口（如 http://192.168.1.100:30080）来测试 Ingress 路由是否生效。

```bash
curl http://172.20.113.158:80
```

```html
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #3e169d;">

<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
    <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547053817/error_404.png">

</div>

</body>
```

跳过证书验证（仅限测试/开发环境！），使用https协议 去访问ingress-nginx 服务

```bash
curl --insecure https://172.20.113.158:443
```

```html
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #3e169d;">

<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
    <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547053817/error_404.png">

</div>

</body>
```

```
404-ERROR-PAGE
```

### 13. Check the Solution

```
OK
```

### 14. Check the Solution

```bash
kubectl edit ingress --namespace app-space
```

Change the path from /watch to /stream

OR

```yaml
apiVersion: v1
items:
- apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    name: ingress-wear-watch
    namespace: app-space
  spec:
    rules:
    - http:
        paths:
        - backend:
            serviceName: wear-service
            servicePort: 8080
          path: /wear
          pathType: ImplementationSpecific
        - backend:
            serviceName: video-service
            servicePort: 8080
          path: /stream
          pathType: ImplementationSpecific
  status:
    loadBalancer:
      ingress:
      - {}
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

### 15. Check the Solution

```
OK
```

### 16. Check the Solution

```
404 ERROR PAGE
```

### 17. Check the Solution

```
OK
```

### 18. Check the Solution

Run the command `kubectl edit ingress --namespace app-space` and add a new Path entry for the new service.

OR

```yaml
apiVersion: v1
items:
- apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    name: ingress-wear-watch
    namespace: app-space
  spec:
    rules:
    - http:
        paths:
        - backend:
            serviceName: wear-service
            servicePort: 8080
          path: /wear
          pathType: ImplementationSpecific
        - backend:
            serviceName: video-service
            servicePort: 8080
          path: /stream
          pathType: ImplementationSpecific
        - backend:
            serviceName: food-service
            servicePort: 8080
          path: /eat
          pathType: ImplementationSpecific
  status:
    loadBalancer:
      ingress:
      - {}
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

### 19. Check the Solution

```
OK
```

### 20. Check the Solution

```
CRITICAL-SPACE
```

### 21. Check the Solution

```
WEBAPP-PAY
```

### 22. Check the Solution

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: critical-ingress  
  namespace: critical-space # 注意要有命名空间
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
            name: pay-service
            port:
              number: 8282 
```

### 23. Check the Solution

```
OK
```
