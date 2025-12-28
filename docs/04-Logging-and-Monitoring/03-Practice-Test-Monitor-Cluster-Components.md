# Practice Test - Monitor Cluster Components
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-monitor-cluster-components/)
  
Solutions to practice test - monitor cluster components
1.  <details>
    <summary>We have deployed a few PODs running workloads. Inspect it.</summary>

    ```
    kubectl get pods
    ```
    </details>
  
1.  <details>
    <summary>Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.</summary>

    ```
    git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
    ```
    </details>

    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml  
serviceaccount/metrics-server created  
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created  
clusterrole.rbac.authorization.k8s.io/system:metrics-server created  
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created  
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created  
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created  
service/metrics-server created  
deployment.apps/metrics-server created  
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created  
  
1.  <details>
    <summary>Deploy the metrics-server by creating all the components downloaded.</summary>
    
    Run the 'kubectl create -f .' command from within the downloaded repository.
  
    ```
    cd kubernetes-metrics-server
    kubectl create -f .
    ```
    </details>
    
1.  <details>
    <summary>It takes a few minutes for the metrics server to start gathering data.</summary>

    Run the `kubectl top node` command and wait for a valid output.
    
    ```
    kubectl top node
    ```
    </details>
  
1.  <details>
    <summary>Identify the node that consumes the most CPU(cores).</summary>

     Run the `kubectl top node` command

      ```
      kubectl top node
      ```

      Examine the `CPU(cores)` column of the output to get the answer.

      </details>
  
1.  <details>
    <summary>Identify the node that consumes the most Memory(bytes).</summary>
    Run the `kubectl top node` command
  
    ```
    $ kubectl top node
    ```

    Examine the `MEMORY(bytes)` column of the output to get the answer.

    </details>
  
1.  <details>
    <summary>Identify the POD that consumes the most Memory(bytes).</summary>

    Run the `kubectl top pod` command
  
    ```
    kubectl top pod
    ```

    Examine the `MEMORY(bytes)` column of the output to get the answer.

    </details>
  
1.  <details>

    <summary>Identify the POD that consumes the least CPU(cores).</summary>

    Run the `kubectl top pod` command
  
    ```
    kubectl top pod
    ```

    Examine the `CPU(cores)` column of the output to get the answer.

  </details>



```json
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        image: registry.k8s.io/metrics-server/metrics-server:v0.8.0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 10250
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100


```


## 🔍 一、这个命令的作用

部署 **Metrics Server** —— Kubernetes 官方推荐的 **集群级资源指标（CPU/内存使用率）收集器**。

### ✅ 为什么需要它？
- `kubectl top node` 或 `kubectl top pod` 命令依赖 Metrics Server 提供实时资源使用数据。
- **Horizontal Pod Autoscaler（HPA，水平 Pod 自动扩缩容）** 也需要它来获取指标。
- 它是 Kubernetes 资源监控体系的核心组件之一（替代了早期的 Heapster）。

---

## 📦 二、`components.yaml` 中创建了哪些资源？（逐项解释）

你看到的输出正是该 YAML 文件中定义的 Kubernetes 对象：

| 资源 | 作用说明 |
|------|--------|
| `serviceaccount/metrics-server` | 为 Metrics Server Pod 创建专用的服务账户（用于身份认证） |
| `clusterrole: system:aggregated-metrics-reader` | 定义一个集群角色：允许读取“聚合 API”中的指标数据（供 APIService 使用） |
| `clusterrole: system:metrics-server` | 定义主权限：允许 Metrics Server 从 Kubelet 获取节点和 Pod 的资源使用情况 |
| `rolebinding: metrics-server-auth-reader` | 在 `kube-system` 命名空间中，允许 Metrics Server 读取 `extension-apiserver-authentication` ConfigMap（用于认证） |
| `clusterrolebinding: system:auth-delegator` | 允许 Metrics Server 将认证请求委托给主 API Server（用于代理请求） |
| `clusterrolebinding: system:metrics-server` | 将 `system:metrics-server` ClusterRole 绑定到 `metrics-server` ServiceAccount，授予实际权限 |
| `service/metrics-server` | 创建一个 ClusterIP 类型的 Service，暴露 Metrics Server（通常只在集群内部访问） |
| `deployment.apps/metrics-server` | 部署 Metrics Server 应用本身（通常 1 副本，运行在 `kube-system` 命名空间） |
| `apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io` | **最关键！** 将 Metrics Server 注册为 Kubernetes 的 **聚合 API（Aggregated API）**，使得 `kubectl top` 等命令能通过标准 API 路径访问指标 |

> 💡 **聚合 API 是什么？**  
> 它允许第三方服务（如 Metrics Server）将自己的 API 接入 Kubernetes 主 API Server 的路径下（如 `/apis/metrics.k8s.io/v1beta1`），对用户透明。

---

## ⚙️ 三、Metrics Server 如何工作？

1. **Metrics Server Pod 启动**，以 `metrics-server` ServiceAccount 身份运行。
2. 它通过 **安全连接（TLS）** 访问每个节点上的 **Kubelet `/stats/summary` 或 `/metrics/resource` 端点**，拉取 CPU/内存使用数据。
3. 数据被缓存并暴露在自己的 HTTP API 上（如 `/apis/metrics.k8s.io/v1beta1/nodes`）。
4. Kubernetes API Server 通过 **APIService 配置**，将对 `metrics.k8s.io` 的请求**代理**给 Metrics Server。
5. 用户执行 `kubectl top pod` 时：
   ```bash
   kubectl top pod → API Server → /apis/metrics.k8s.io/... → Metrics Server → 返回指标
