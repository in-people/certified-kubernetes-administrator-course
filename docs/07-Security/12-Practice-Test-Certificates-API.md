# Kubernetes 证书 API 实践测试 - 证书签发流程

## 实践测试说明
- 了解 Kubernetes 证书 API 的使用方法
- 参考练习测试：[Practice Test](https://kodekloud.com/topic/practice-test-certificates-api/)

## 问题描述
- 一个新成员 akshay 加入了我们的团队，他需要访问我们的集群。证书签名请求（Certificate Signing Request）位于 `/root` 目录。

<details>
```
$ ls -l /root
```
</details>

## 解决方案
- 参考答案位置：`/var/answers/akshay-csr.yaml`

### Kubernetes CertificateSigningRequest（证书签名请求）对象的配置说明

**metadata.name: akshay**
- 这个 CSR 对象的名称，用于标识

**signerName: kubernetes.io/kube-apiserver-client**
- 非常重要：指定证书的签发者和用途类型

`kubernetes.io/kube-apiserver-client` 表示：
- 证书将由 Kubernetes 集群的 API 服务器客户端 CA 签发
- 这个证书用于客户端身份验证（而不是服务器证书）
- 通常用于 kubectl 用户认证、服务账户认证等

**usages: - client auth**
- 指定证书的用途
- `client auth` 表示这个证书用于客户端身份验证

具体用途包括：
- 用户使用 kubectl 访问集群
- Pod 中的应用程序访问 API 服务器
- 其他需要认证的客户端操作

---

## YAML 配置示例

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: <Paste the base64 encoded value of the CSR file>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

<details>
```
$ kubectl create -f /var/answers/akshay-csr.yaml
```
</details>

## 操作步骤

### 1. 查看 CSR 状态
- 执行命令 `kubectl get csr`

<details>
```
$ kubectl get csr
```
</details>

输出示例：
```
kubectl get CertificateSigningRequest
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      13s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Pending
```

### 2. 批准 CSR 请求
- 执行命令 `kubectl certificate approve akshay`

<details>
```
$ kubectl certificate approve akshay
```
</details>

### 3. 再次查看 CSR 状态
- 执行命令 `kubectl get csr`

<details>
```
$ kubectl get csr
```
</details>

### 4. 检查请求者信息
- 执行命令 `kubectl get csr` 并查看 Requestor 列

<details>
```
$ kubectl get csr
```
</details>

## 补充说明
- 其他 CSR 是在 TLS Bootstrapping 过程中请求的。我们将在课程的 TLS bootstrap 部分进一步讨论。

### 其他 CSR 操作示例

#### 1. 查看 CSR 详细信息
- 执行命令 `kubectl get csr agent-smith -o yaml`

<details>
```
$ kubectl get csr agent-smith -o yaml
```
</details>

#### 2. 拒绝 CSR 请求
- 执行命令 `kubectl certificate deny agent-smith`

<details>
```
$ kubectl certificate deny agent-smith
```
</details>

#### 3. 删除 CSR
- 执行命令 `kubectl delete csr agent-smith`

<details>
```
$ kubectl delete csr agent-smith
```
</details>
