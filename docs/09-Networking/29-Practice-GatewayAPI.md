# Gateway API 实践练习

## 问题1：

What is the purpose of the allowedRoutes field in a Gateway?
（Listener）或网关（Gateway）上，从而实现对流量入口的精细化权限管理和安全隔离。

How does a GatewayClass differ from a Gateway?

How does a GatewayClass differ from a Gateway?
（使用 Gateway API 相比 Ingress 的主要优势是什么？）

It supports more advanced routing and multi-protocol support.
（它支持更高级的路由和多协议支持。）

To use the Gateway API, a controller is required. In this lab, we will install NGINX Gateway Fabric as the controller. Follow these steps to complete the installation:

## 问题2：

Create a Kubernetes Gateway resource with the following specifications:

- Name: nginx-gateway
- Namespace: nginx-gateway
- Gateway Class Name: nginx
- Listeners:
  - Protocol: HTTP
  - Port: 80
  - Name: http
- Allowed Routes: All namespaces


### Install the Gateway API resources

```bash
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | kubectl apply -f -
```

### Deploy the NGINX Gateway Fabric CRDs

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
```

### Deploy NGINX Gateway Fabric

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml
```

### Verify the Deployment

```bash
kubectl get pods -n nginx-gateway
```

### View the nginx-gateway service

```bash
kubectl get svc -n nginx-gateway nginx-gateway -o yaml
```

### Update the nginx-gateway service to expose ports 30080 for HTTP and 30081 for HTTPS

```bash
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'
```

Prerequisite: Make sure the Gateway API CRDs are installed and a compatible controller (like NGINX Gateway Fabric) is running.

### Create the Gateway manifest (gateway.yaml):

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      port: 80
      protocol: HTTP
      allowedRoutes: 
        namespaces: 
          from: All
```

### Deploy the manifest:

```bash
kubectl apply -f gateway.yaml
```

To verify the succesfull deployment, run the commands below:

```bash
kubectl get gateways -n nginx-gateway
kubectl describe gateway nginx-gateway -n nginx-gateway
```

## 问题3：

A new pod named frontend-app and a service called frontend-svc have been deployed in the default namespace.
Expose the service on the / path by creating an HTTPRoute named frontend-route.

### Confirm the pod and service exist:

```bash
kubectl get pod,svc -n default
```

### Create the HTTPRoute manifest:

Create a file named frontend-route.yaml with the following content:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: frontend-route
  namespace: default
spec:
  parentRefs:
    - name: nginx-gateway           # Name of the Gateway
      namespace: nginx-gateway      # Namespace where the Gateway is deployed
      sectionName: http             # Attach to the 'http' listener
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: frontend-svc
          port: 80
```

parentRefs attaches the route to the nginx-gateway Gateway in the nginx-gateway namespace, specifically to the http listener.
The rule matches all requests with a path prefix of / and forwards them to the frontend-svc service on port 80.

### Apply the manifest:

```bash
kubectl apply -f frontend-route.yaml
```

### Verify the HTTPRoute:

```bash
kubectl get httproute frontend-route 
kubectl describe httproute frontend-route
```
