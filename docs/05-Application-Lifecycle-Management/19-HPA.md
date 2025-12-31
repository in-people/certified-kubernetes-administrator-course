# Kubernetes HPA (Horizontal Pod Autoscaler) 配置与事件分析

## 1. HPA 配置

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: nginx-deployment
spec:
  maxReplicas: 3
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 80
        type: Utilization
    type: Resource
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
status:
  currentMetrics: null
  desiredReplicas: 0
  currentReplicas: 0
```

## 2. 问题说明

### 2.1 HPA 主要作用
- 自动化扩缩容 Pod，基于观测到的 CPU 使用率或其他选定指标

### 2.2 指标提供组件
- `metrics-server` 负责向 HPA 提供指标数据

### 2.3 常见问题
- 当 HPA 状态显示 CPU 目标为 `/80` 时，通常表示指标服务器不可用或运行不正常

## 3. Deployment 配置

### 3.1 原始配置（缺少资源请求）
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}  # 注意：这里缺少资源请求和限制
```

### 3.2 修复后的配置（添加了资源请求）
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 7
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
         requests:
           cpu: 100m
         limits:
           cpu: 200m
```

## 4. HPA 事件日志

### 4.1 扩缩容事件
以下显示了 Deployment 的副本数变化历史：

```
kubectl events hpa nginx-deployment |  grep -i "ScalingReplicaSet"

24m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-bf744486c from 0 to 7
20m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 7 to 3
11m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-bf744486c from 3 to 7
11m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 6 to 3
11m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 7 to 6
11m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-6c78464d7c from 0 to 2
11m                 Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 3 to 2
10m (x2 over 11m)   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-6c78464d7c from 2 to 3
10m (x3 over 11m)   Normal    ScalingReplicaSet              Deployment/nginx-deployment                (combined from similar events): Scaled down replica set nginx-deployment-bf744486c from 1 to 0
10m (x2 over 11m)   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-6c78464d7c from 3 to 1
```

### 4.2 错误事件
以下显示了 HPA 无法获取 CPU 指标的错误：

```
kubectl events hpa nginx-deployment | grep -i "FailedGetResourceMetric"
24m (x3 over 24m)   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-jxwsl
23m (x8 over 26m)   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-bbzcc
21m (x4 over 25m)   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-2wtvb
```

## 5. 问题分析与解决

### 5.1 问题原因
HPA 无法获取 CPU 使用率指标，错误原因是 Pod 中的容器没有设置 CPU 资源请求（missing request for cpu）。

### 5.2 解决方案
需要在 nginx-deployment 的 Pod 模板中添加 CPU 资源请求，如修复后的配置所示。

### 5.3 影响
没有设置资源请求会导致 HPA 无法计算 CPU 利用率百分比，从而无法正常进行自动扩缩容。
