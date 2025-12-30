# Admission Controller

**问题：** 以下哪项不是**准入控制器**（admission controller）的功能？

**选项：**

* validate configuration（验证配置）
* perform additional operations before the pod gets created（在 Pod 创建之前执行额外操作）
* help us implement better security measures（帮助我们实施更好的安全措施）

**问题：** 哪个准入控制器默认未启用？

**选项：**

* ValidatingAdmissionWebhook  默认启用。用于调用外部 webhook 进行配置验证。
* NamespaceLifecycle   默认启用。确保命名空间的生命周期管理（如防止删除系统命名空间）。
* MutatingAdmissionWebhook  默认启用。用于在资源创建前修改其内容（如注入 sidecar）。

通过查看apiserver的配置，查看--enable-admission  cat /etc/kubernetes/manifests/kube-apiserver.yaml

比如执行 kubectl run nginx --image nginx -n blue 报 Error from server (NotFound): namespaces "blue" not found

如果配置了，--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision，那么

kubectl run nginx --image nginx -n blue 会自动创建blue namespace

目前使用NamespaceLifecycle

查看apiserver准入插件的配置

```bash
ps -ef | grep kube-apiserver | grep admission-plugins
```

**First Mutating then Validating** （先执行 Mutating，再执行 Validating）

---

**解释：**

在 Kubernetes 中，准入控制器的执行流程是：

1. **Mutating Admission Controllers（变更型准入控制器）** 会首先被调用，用于修改资源对象（例如自动注入 sidecar、设置默认值等）。
2. 然后是 **Validating Admission Controllers（验证型准入控制器）**，它们对已修改后的资源进行验证，确保其符合策略。

这个顺序非常重要，因为验证是在变更之后进行的，以确保所有修改都符合规则。

## 什么是 Webhook？

### Webhook 的含义

**Webhook** 是一种基于 HTTP 的回调机制。简单来说，就是一个应用程序在特定事件发生时，主动向另一个预先配置的 URL 发送 HTTP 请求（POST），通知对方发生了什么事件，让对方可以做出相应的处理。

可以把 Webhook 理解为：

- **事件订阅**：你订阅了某些事件，当这些事件发生时，系统会主动通知你
- **HTTP 回调**：通过 HTTP 请求把信息"推"给你，而不是你去"拉"取

### Kubernetes Admission Webhook 的作用

在 Kubernetes 中，**Admission Webhook** 允许**扩展准入控制逻辑**，实现自定义的业务规则和安全策略。

#### 1. 两种类型的 Webhook

**MutatingAdmissionWebhook（变更型）**

- 在资源创建/更新**之前**修改对象
- 例如：自动注入 sidecar 容器、设置默认值、添加标签

**ValidatingAdmissionWebhook（验证型）**

- 在资源创建/更新之前验证对象
- 例如：检查镜像来源、验证资源配额、强制安全策略

#### 2. 工作流程

```
用户请求 (kubectl apply)
    ↓
认证 & 授权
    ↓
Mutating Webhook ← 可能修改资源（如自动添加 securityContext）
    ↓
Validating Webhook ← 验证修改后的资源（如检查是否以 root 运行）
    ↓
持久化到 etcd
```

这个顺序非常重要：**先执行 Mutating，再执行 Validating**。因为验证是在变更之后进行的，以确保所有修改都符合规则。

#### 3. 实际应用场景

本节后续文档中的 webhook 实现了以下安全控制：

- **拒绝未设置 securityContext 的 Pod 以 root 运行**
- **自动为 Pod 添加 `runAsNonRoot: true` 和 `runAsUser: 1234`**
- **当检测到冲突配置（如要求非 root 但设置 uid=0）时直接拒绝请求**

这样就不需要在每个 Pod 配置中手动设置安全上下文，webhook 会自动处理，实现了集群级别的策略自动化。

## 什么是 SecurityContext？

### SecurityContext 的含义

**SecurityContext** 是 Kubernetes 中用于定义 Pod 或容器**安全配置**的机制。它控制容器进程的权限和访问控制，是容器安全的核心配置项。

### 为什么需要 SecurityContext？

默认情况下，容器内的进程可能以 **root 用户**运行，这存在安全风险：

- 如果容器被攻击，攻击者获得 root 权限
- 可能访问宿主机敏感资源
- 违反最小权限原则

### SecurityContext 的两个层级

#### 1. Pod 级别（对所有容器生效）

```yaml
spec:
  securityContext:
    runAsNonRoot: true    # 强制非 root 用户运行
    runAsUser: 1234       # 指定用户 ID
    fsGroup: 2000         # 文件系统组 ID
```

#### 2. Container 级别（只对单个容器生效，优先级更高）

```yaml
spec:
  containers:
  - name: myapp
    securityContext:
      runAsNonRoot: true
      runAsUser: 1234
      allowPrivilegeEscalation: false  # 禁止权限提升
      readOnlyRootFilesystem: true      # 根文件系统只读
      capabilities:
        drop: ["ALL"]    # 移除所有 Linux 能力
        add: ["NET_BIND_SERVICE"]  # 只添加必要的
```

### 常用配置项说明

| 配置项                       | 说明                 | 推荐值             |
| ---------------------------- | -------------------- | ------------------ |
| `runAsNonRoot`             | 禁止以 root 用户运行 | `true`           |
| `runAsUser`                | 指定运行用户 ID      | > 0 的非 root 用户 |
| `allowPrivilegeEscalation` | 是否允许权限提升     | `false`          |
| `readOnlyRootFilesystem`   | 根文件系统只读       | `true`           |
| `capabilities.drop`        | 移除 Linux 能力      | `["ALL"]`        |
| `seccompProfile`           | 系统调用过滤         | `RuntimeDefault` |

### Webhook 如何使用 SecurityContext

在本节文档的示例中，webhook 的作用就是**自动为 Pod 添加 SecurityContext 配置**：

```yaml
# 原始 Pod（没有 securityContext）
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-defaults
spec:
  containers:
  - name: busybox
    image: busybox

# 经过 MutatingWebhook 修改后
spec:
  securityContext:           # ← webhook 自动添加
    runAsNonRoot: true       # ← 强制非 root
    runAsUser: 1234          # ← 使用指定用户
  containers:
  - name: busybox
    image: busybox
```

这样即使用户忘记配置安全上下文，webhook 也会自动添加，确保集群安全。

### 完整安全配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: myapp
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

**SecurityContext 是 Kubernetes 容器安全的第一道防线。**

---

myclaim.yaml 创建以下PVC，会自动添加 storageClassName: default

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 0.5Gi
```

查看创建好的myclaim pvc

```bash
controlplane ~ ➜  kubectl get pvc
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
myclaim   Pending                                      default        `<unset>`                 20s
```

```yaml
controlplane ~ ➜  kubectl get  pvc myclaim -oyaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"myclaim","namespace":"default"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"0.5Gi"}}}}
  creationTimestamp: "2025-12-30T10:31:25Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: myclaim
  namespace: default
  resourceVersion: "1575"
  uid: dabbffcf-dbc7-4f41-9ecd-79aea53a5474
spec:
  accessModes:
  - ReadWriteOnce
  resources:
  requests:
  storage: 512Mi
  storageClassName: default
```

添加 `--disable-admission-plugins=DefaultStorageClass`

要**禁用 DefaultStorageClass 准入控制器**。实施此更改后，Kubernetes 将不再为未显式指定 storageClassName 的新 PersistentVolumeClaim（PVC）自动分配默认的 StorageClass。

再次部署，则不会添加默认storageclass，STORAGECLASS为空

```bash
kubectl get pvc myclaim
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
myclaim   Pending                                                     `<unset>`                 6s
```

查看apiserver准入插件的配置

```bash
ps -ef | grep kube-apiserver | grep admission-plugins
```

**First Mutating then Validating** （先执行 Mutating，再执行 Validating）

---

**解释：**

在 Kubernetes 中，准入控制器的执行流程是：

1. **Mutating Admission Controllers（变更型准入控制器）** 会首先被调用，用于修改资源对象（例如自动注入 sidecar、设置默认值等）。
2. 然后是 **Validating Admission Controllers（验证型准入控制器）**，它们对已修改后的资源进行验证，确保其符合策略。

这个顺序非常重要，因为验证是在变更之后进行的，以确保所有修改都符合规则。

### 创建 TLS Secret

```bash
kubectl -n webhook-demo create secret tls webhook-server-tls \
    --cert "/root/keys/webhook-server-tls.crt" \
    --key "/root/keys/webhook-server-tls.key"
```

#### 1. `kubectl`

Kubernetes 的命令行工具，用于与集群交互。

#### 2. `-n webhook-demo`

指定操作的目标命名空间为 **`webhook-demo`**。

> 如果省略 `-n`，默认使用当前上下文配置的命名空间（通常是 `default`）。

#### 3. `create secret tls webhook-server-tls`

* 创建一个类型为 **`kubernetes.io/tls`** 的 Secret。
* Secret 的名称为 **`webhook-server-tls`**。
* 这种类型的 **Secret 专门用于存储 TLS 证书（`tls.crt`）和私钥（`tls.key`），常用于 HTTPS 服务、Webhook 服务器**等场景。

#### 4. `--cert "/root/keys/webhook-server-tls.crt"`

指定本地文件路径中的 **TLS 证书文件**（公钥证书），该文件内容将被存入 Secret 的 `tls.crt` 字段。

#### 5. `--key "/root/keys/webhook-server-tls.key"`

指定本地文件路径中的 **TLS 私钥文件**，该文件内容将被存入 Secret 的 `tls.key` 字段。

---

### 最终效果：

在命名空间 `webhook-demo` 中创建如下 Secret：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: webhook-server-tls
  namespace: webhook-demo
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded certificate>
  tls.key: <base64-encoded private key>
```

---

### 典型用途：

这个 Secret 通常会被挂载到 **Admission Webhook** 或 **Custom API Server** 的 Pod 中，供其启用 HTTPS 服务，以便 Kubernetes API Server 能通过 TLS 安全地调用该 Webhook。

> ⚠️ 注意：证书必须由 API Server 信任的 CA 签发（或使用自签名证书并将其 CA 配置到 WebhookConfiguration 中）。

### 下面的webhook会使用这个secret

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webhook-server
  namespace: webhook-demo
  labels:
    app: webhook-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-server
  template:
    metadata:
      labels:
        app: webhook-server
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1234
      containers:
      - name: server
        image: stackrox/admission-controller-webhook-demo:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8443
          name: webhook-api
        volumeMounts:
        - name: webhook-tls-certs
          mountPath: /run/secrets/tls
          readOnly: true
      volumes:
      - name: webhook-tls-certs
        secret:
          secretName: webhook-server-tls
```

### 创建一个service

```yaml
cat webhook-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webhook-server
  namespace: webhook-demo
spec:
  selector:
    app: webhook-server
  ports:
    - port: 443
      targetPort: webhook-api
```

### Webhook Configuration

```yaml
cat webhook-configuration.yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
- name: webhook-server.webhook-demo.svc
  clientConfig:
    service:
      name: webhook-server
      namespace: webhook-demo
      path: "/mutate"
    caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQekNDQWllZ0F3SUJBZ0lVTFR5b1NBL2ZkZWNQZG9yMDd0S3JtU2ZaT1hNd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0x6RXRNQ3NHQTFVRUF3d2tRV1J0YVhOemFXOXVJRU52Ym5SeWIyeHNaWElnVjJWaWFHOXZheUJFWlcxdgpJRU5CTUI0WERUSTFNVEl6TURFd05EUXpORm9YRFRJMk1ERXlPVEV3TkRRek5Gb3dMekV0TUNzR0ExVUVBd3drClFXUnRhWE56YVc5dUlFTnZiblJ5YjJ4c1pYSWdWMlZpYUc5dmF5QkVaVzF2SUVOQk1JSUJJakFOQmdrcWhraUcKOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXZDNGtpcTN6M2Q4YlNWZUpBRVI4ZVowRXM5ZmJnbTZzZHEzRQptbVZ1SnNvRS9zTHBWc1FlS2hwN2NleFJTelJMNmxZTVllNHNJa1AyOS82aFZ4RzRsanI4WkNabWthYlRmSlpmCm5OZGpsL2VTZ3lkS1hlN0VwMDA4NkxvTE1TckhzZ1BWNVhuWFFDVXRmOVBjaEx6R2hMUDZQMWxZS3FZV2xOc1kKdjI3VlNBMVlIV1VFYmNIZW53c3RNSTdZZXYyZ2RObkhkd2VVbkpiNEpEbzJCZ0FYS3dtTkZVMWh2Q21QbFFKSwo5cVVReTJCOTNMMi85eEFGSGczckM2Z0xReVRQQW53SGlHOVFxdk9WanNsbEorY3NLSUEyR0NPbC92dWNYU2NoCmtSM1VReVZ4emVMYU02MlAvR0gvcWxkWEordGFObzZpeUF0SXdGeHNYYVpReFBEcjJ3SURBUUFCbzFNd1VUQWQKQmdOVkhRNEVGZ1FVd0ZTdWtxUDQyOFJLcGpuL2hJL3ltc3pIYk1Nd0h3WURWUjBqQkJnd0ZvQVV3RlN1a3FQNAoyOFJLcGpuL2hJL3ltc3pIYk1Nd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DCkFRRUFJQnlDNitPK2NIR2F3SmNUcDdYeWtKS0hWbjBibWRyZ0RhNDA1bCs2dFJMUHp3RkdNZ0lOc1BjL25Bc3UKa0pKNENFUFhVMEFYK29INGUxV0gvNVM5TGtsejZ0d2JNSk1JWHh2bFc2ZmFBVE9QbUxHOTd6a01BKzFaVFhNNApwcmNKZy9Nb3Z3ME14ZFdZUHJPcFhrZUJzTFliSm9VSnM3dFFmWHIyZ1pyb2tPQVN2Qk5nVnhMd0RjSnRxTzZjCkdQSVoyZGhjUTJzVGlZc2Y5UWpaQUpFeWZodVQ3S1A2U0EwcGE0MWtqeTJXSlBlZFBhbWpyTzIyY0dVayt1bk8KL3pOL3J1eDV1c1ZZVkExYmxyZ0tXTmtZQXBjY1BGV0hiNWhlT0lvZjAyMy9ZcmN4L1VvT3cvWUpPTmpHZ0p5Vwo4WFFLWHM2N08wTkorbC9BcGdQcEFnN0Vjdz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  rules:
  - operations: ["CREATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
    admissionReviewVersions: ["v1beta1"]
    sideEffects: None
```

#### MutatingWebhookConfiguration 配置详解

这是一个 **MutatingWebhookConfiguration** 配置文件，用于定义变更型准入 Webhook 的行为。

##### 整体结构

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration  # 资源类型：变更型 Webhook 配置
metadata:
  name: demo-webhook                 # 配置名称
webhooks:
- name: webhook-server.webhook-demo.svc  # Webhook 名称（必须是 DNS 子域名格式）
```

##### 核心配置字段

**1. clientConfig - Webhook 服务器连接配置**

```yaml
clientConfig:
  service:
    name: webhook-server              # Kubernetes Service 名称
    namespace: webhook-demo           # Service 所在的命名空间
    path: "/mutate"                   # HTTP 请求路径
```

**作用**：告诉 Kubernetes API Server 如何调用 Webhook 服务

- 当有符合条件的资源请求时，API Server 会发送请求到：
  ```
  https://webhook-server.webhook-demo.svc/mutate
  ```

**2. caBundle - CA 证书（Base64 编码）**

```yaml
caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQ...
```

**作用**：

- 这是 PEM 格式的 CA 证书的 Base64 编码
- 用于验证 Webhook 服务器的 TLS 证书
- API Server 使用这个 CA 来确保连接到的是合法的 Webhook 服务器

**如何生成**：

```bash
# 生成 CA 证书并编码
cat ca.crt | base64 | tr -d '\n'
```

**3. rules - 触发规则**

```yaml
rules:
- operations: ["CREATE"]              # 监听的操作类型
  apiGroups: [""]                     # API 组（"" = core 组）
  apiVersions: ["v1"]                 # API 版本
  resources: ["pods"]                 # 资源类型
```

| 字段            | 值             | 说明                                                                         |
| --------------- | -------------- | ---------------------------------------------------------------------------- |
| `operations`  | `["CREATE"]` | 只监听**创建**操作。可以是 `["CREATE", "UPDATE", "DELETE"]`          |
| `apiGroups`   | `[""]`       | 核心 API 组（Pod、Service 等）。`["apps"]` 表示 Deployment、StatefulSet 等 |
| `apiVersions` | `["v1"]`     | API 版本                                                                     |
| `resources`   | `["pods"]`   | 只对**Pod** 资源生效                                                   |

**实际效果**：

- ✅ 当用户创建 Pod 时 → Webhook 被调用
- ❌ 当用户更新 Pod 时 → Webhook **不**被调用
- ❌ 当用户创建 Deployment 时 → Webhook **不**被调用

**4. admissionReviewVersions**

```yaml
admissionReviewVersions: ["v1beta1"]
```

**作用**：告诉 API Server 发送哪种版本的 AdmissionReview 对象给 Webhook

**5. sideEffects**

```yaml
sideEffects: None
```

**作用**：告诉 Kubernetes 这个 Webhook 是否有副作用

- `None`：无副作用（推荐，允许并行处理）
- `Some`：可能有副作用

##### 完整工作流程

```
1. 用户执行：kubectl apply -f pod.yaml

2. API Server 收到请求，发现是 CREATE Pod 操作

3. 检查 MutatingWebhookConfiguration，发现匹配规则

4. API Server 构造 AdmissionReview 请求：
   POST https://webhook-server.webhook-demo.svc/mutate
   Body: {
     "request": {
       "uid": "xxx",
       "kind": {"kind": "Pod"},
       "object": <Pod 的 YAML>
     }
   }

5. Webhook 服务器接收请求，修改 Pod 对象（添加 securityContext）

6. Webhook 返回响应：
   {
     "response": {
       "uid": "xxx",
       "allowed": true,
       "patch": <JSON Patch 格式的修改>
     }
   }

7. API Server 应用 patch，将修改后的 Pod 持久化到 etcd
```

##### 与文档示例的关系

这个配置实现了：

```yaml
# 用户创建的 Pod（原始）
spec:
  containers:
  - name: busybox
    image: busybox

# Webhook 修改后的 Pod
spec:
  securityContext:              # ← Webhook 添加
    runAsNonRoot: true
    runAsUser: 1234
  containers:
  - name: busybox
    image: busybox
```

##### 生产环境配置建议

```yaml
webhooks:
- name: webhook-server.webhook-demo.svc
  clientConfig:
    service:
      name: webhook-server
      namespace: webhook-demo
      path: "/mutate"
    caBundle: <base64-encoded-ca>
  rules:
  - operations: ["CREATE", "UPDATE"]  # 监听创建和更新
    apiGroups: ["*"]                  # 所有 API 组（谨慎）
    resources: ["pods", "deployments"]  # 多种资源
    apiVersions: ["v1"]
  admissionReviewVersions: ["v1"]
  sideEffects: None
  timeoutSeconds: 5                  # Webhook 响应超时时间
  failurePolicy: Fail                # Fail: 拒绝请求（安全优先）
                                    # Ignore: 忽略 Webhook（可用性优先）
  namespaceSelector:                 # 只在特定命名空间生效
    matchLabels:
      environment: production
```

### Webhook 行为说明

在前面的步骤中，您已经设置并部署了一个演示用的 webhook，其行为如下：

* **拒绝所有请求**：如果 Pod 的容器未提供 `securityContext`，则禁止以 root 用户运行。
* **默认设置**：如果未设置 `runAsNonRoot`，webhook 会自动添加 `runAsNonRoot: true`，并将用户 ID 设置为 1234。
* **显式允许 root 访问**：只有当您在 Pod 的 `securityContext` 中显式设置 `runAsNonRoot: false` 时，webhook 才允许容器以 root 用户运行。

在接下来的步骤中，您将看到针对上述每种场景的 Pod 定义文件。请使用提供的定义文件部署这些 Pod，并验证我们 webhook 的行为是否符合预期。

### Pod 示例 1: pod-with-defaults

```yaml
cat pod-with-defaults.yaml

# A pod with no securityContext specified.
# Without the webhook, it would run as user root (0). The webhook mutates it
# to run as the non-root user with uid 1234.

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-defaults
  labels:
    app: pod-with-defaults
spec:
  restartPolicy: OnFailure
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```

验证 securityContext 已被自动添加：

```bash
kubectl get pod -o yaml | grep -5 securityContext
    nodeName: controlplane
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: OnFailure
    schedulerName: default-scheduler
    securityContext:
      runAsNonRoot: true
      runAsUser: 1234
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
```

### Pod 示例 2: pod-with-override

```yaml
controlplane ~ ➜  kubectl apply -f pod-with-override.yaml
pod/pod-with-override created

controlplane ~ ➜  cat pod-with-override.yaml

# A pod with a securityContext explicitly allowing it to run as root.
# The effect of deploying this with and without the webhook is the same. The
# explicit setting however prevents the webhook from applying more secure
# defaults.

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-override
  labels:
    app: pod-with-override
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: false
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```



验证 pod-with-override 的配置保持不变：

```bash
controlplane ~ ➜  kubectl get pod pod-with-override -oyaml | grep  -5 securityContext
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      ,"labels":,"name":"pod-with-override","namespace":"default"},"spec":],"restartPolicy":"OnFailure","securityContext":}}
  creationTimestamp: "2025-12-30T11:10:54Z"
  generation: 1
  labels:
    app: pod-with-override
  name: pod-with-override
-------------------------

  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: OnFailure
  schedulerName: default-scheduler
  securityContext:
    runAsNonRoot: false
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:

controlplane ~ ➜
```


### Pod 示例 3: pod-with-conflict

```yaml
cat pod-with-conflict.yaml

# A pod with a conflicting securityContext setting: it has to run as a non-root
# user, but we explicitly request a user id of 0 (root).
# Without the webhook, the pod could be created, but would be unable to launch
# due to an unenforceable security context leading to it being stuck in a
# 'CreateContainerConfigError' status. With the webhook, the creation of
# the pod is outright rejected.

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-conflict
  labels:
    app: pod-with-conflict
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: true
    runAsUser: 0
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
```

创建冲突的 Pod 会被拒绝：

```bash
controlplane ~ ➜  kubectl apply -f  pod-with-conflict.yaml
Error from server: error when creating "pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)
```

验证 pod-with-override 的配置保持不变：

```bash
controlplane ~ ➜  kubectl get pod pod-with-override -oyaml | grep  -5 securityContext
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      ,"labels":,"name":"pod-with-override","namespace":"default"},"spec":],"restartPolicy":"OnFailure","securityContext":}}
  creationTimestamp: "2025-12-30T11:10:54Z"
  generation: 1
  labels:
    app: pod-with-override
  name: pod-with-override
-------------------------

  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: OnFailure
  schedulerName: default-scheduler
  securityContext:
    runAsNonRoot: false
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:

controlplane ~ ➜
```
