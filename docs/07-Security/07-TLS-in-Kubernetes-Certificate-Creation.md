# TLS in kubernetes - Certificate Creation
  - Take me to [Video Tutorial](https://kodekloud.com/topic/tls-in-kubernetes-certificate-creation/)
  
In this section, we will take a look at TLS certificate creation in kubernetes

## Generate Certificates
- There are different tools available such as easyrsa, openssl or cfssl etc. or many others for generating certificates.
- 有多种可用工具可用于生成证书，例如 easyrsa、openssl 或 cfssl 等众多其他工具。

## Certificate Authority (CA)

- Generate Keys
  ```
  $ openssl genrsa -out ca.key 2048
  ```
目的：创建证书颁发机构（CA）的私钥文件。   
参数解析：   
genrsa：生成 RSA 私钥的命令。   
-out ca.key：将生成的私钥保存到 ca.key 文件中。   
2048：指定密钥长度为 2048 位（当前的安全标准）。   
重要说明： 
ca.key 是核心机密文件，必须严格保护。任何人获取此文件都可以签发伪造证书。  
此私钥将用于后续为 CA 自身签发证书，并为其他证书签名。  

- Generate CSR
  ```
  $ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
  ```

目的：创建一个证书签名请求（CSR），用于申请 CA 证书。  
参数解析：  
req -new：创建一个新的证书请求。  
-key ca.key：指定使用上一步生成的私钥 ca.key 来生成 CSR。  
-subj "/CN=KUBERNETES-CA"：设置证书的主题（Subject），这里将通用名称（CN）设置为 KUBERNETES-CA。这将是此 CA 的身份标识。  
-out ca.csr：将 CSR 保存到 ca.csr 文件中。  
关键点：CSR 本身不是证书，而是一个包含公钥和身份信息的请求文件，需要被签名才能成为有效证书。  

- Sign certificates
  ```
  $ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  ```

目的：使用 CA 自身的私钥对 CSR 进行自签名，生成最终的 CA 根证书。  
参数解析：  
x509 -req：处理证书请求并输出 X.509 格式的证书。  
-in ca.csr：指定输入的 CSR 文件。  
-signkey ca.key：指定用于签名的私钥（这里是 CA 自己的私钥）。  
-out ca.crt：输出生成的 CA 根证书文件。  

核心逻辑：  
这是一个自签名（Self-Signed） 过程。因为这是根 CA，没有上一级机构为其签名，所以用自己的私钥为自己签名。  
生成的 ca.crt 是公钥证书，将分发给所有需要验证由该 CA 签发证书的客户端或服务（例如，所有 Kubernetes 组件）。  

CA 证书文件

| 文件 | 类型 | 用途 | 关键性 |
|------|------|------|------|
| ca.key | 私钥 | CA 的私钥，用于为其他证书签名 | 绝密，必须离线安全保存 |
| ca.csr | 证书签名请求 | 生成 CA 证书的中间请求文件（通常可删除） | 临时文件，生成证书后不再需要 |
| ca.crt | 公钥证书（根证书） | CA 的根证书，用于验证所有由该 CA 签发的证书 | 公开分发，需部署到所有信任该 CA 的实体 |

TLS 证书工作原理

TLS（Transport Layer Security，传输层安全协议）是一种加密协议，用于在互联网通信中提供安全连接。TLS 证书体系基于公钥基础设施（PKI），通过证书颁发机构（CA）来建立信任链。

证书签发流程

1. **创建 CA**：首先创建根证书颁发机构（Root CA），包括私钥（ca.key）和根证书（ca.crt）
2. **生成 CSR**：需要证书的实体生成私钥和证书签名请求（CSR）
3. **签发证书**：CA 使用其私钥对 CSR 进行签名，生成证书
4. **部署证书**：将证书和私钥部署到需要安全通信的服务上
 
 ![ca1](../../images/ca1.PNG)
 
## Generating Client Certificates

#### Admin User Certificates

- Generate Keys
  ```
  $ openssl genrsa -out admin.key 2048
  ```
- Generate CSR
  ```
  $ openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
  ```
- Sign certificates
  ```
  $ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
  ```
  
  ![ca2](../../images/ca2.PNG)
  
- Certificate with admin privilages
  ```
  $ openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
  ```
  
#### We follow the same procedure to generate client certificate for all other components that access the kube-apiserver.

  ![crt1](../../images/crt1.PNG)
  
  ![crt2](../../images/crt2.PNG)
  
  ![crt3](../../images/crt3.PNG)
   
  ![crt4](../../images/crt4.PNG)
  
## Generating Server Certificates

## ETCD Server certificate

  ![etc1](../../images/etc1.PNG)
  
  ![etc2](../../images/etc2.PNG)
  
## Kube-apiserver certificate

  ![api1](../../images/api1.PNG)
  
  ![api2](../../images/api2.PNG)
  
## Kubectl Nodes (Server Cert)

   ![kctl1](../../images/kctl1.PNG)
   
## Kubectl Nodes (Client Cert)

   ![kctl2](../../images/kctl2.PNG)
   
# Kubernetes API Server 安全配置参数说明

## 参数列表

```yaml
- --tls-cert-file=/etc/kubernetes/certs/server.crt
- --tls-private-key-file=/etc/kubernetes/certs/server.key
- --client-ca-file=/etc/kubernetes/certs/ca.crt
- --service-account-key-file=/etc/kubernetes/certs/sign_sa_pub.key
- --service-account-signing-key-file=/etc/kubernetes/certs/sign_sa_pri.key
- --service-account-issuer=https://kubernetes.default.svc.cluster.local
- --etcd-cafile=/etc/kubernetes/certs/etcd/Etcdtrustca.crt
- --etcd-certfile=/etc/kubernetes/certs/etcd/etcd-client.crt
- --etcd-keyfile=/etc/kubernetes/certs/etcd/etcd-client.key
- --requestheader-client-ca-file=/etc/kubernetes/certs/proxy-ca.crt
- --proxy-client-cert-file=/etc/kubernetes/certs/proxy.crt
- --proxy-client-key-file=/etc/kubernetes/certs/proxy.key
- --kubelet-certificate-authority=/etc/kubernetes/certs/apiserver_to_kubelet_cert/ca.crt
- --kubelet-client-certificate=/etc/kubernetes/certs/apiserver_to_kubelet_cert/client.crt
- --kubelet-client-key=/etc/kubernetes/certs/apiserver_to_kubelet_cert/client.key
```

---

## 详细配置说明

### 1. API Server 本身的 TLS 配置（对外提供 HTTPS 服务）

**参数：**
```bash
--tls-cert-file=/etc/kubernetes/certs/server.crt
--tls-private-key-file=/etc/kubernetes/certs/server.key
```

**作用：** 指定 API Server 对外提供 HTTPS 服务所使用的服务器证书和私钥。

**说明：** 客户端（如 kubectl、kubelet、其他组件）通过这个证书验证 API Server 的身份。

---

### 2. 客户端身份认证（Client Authentication）

**参数：**
```bash
--client-ca-file=/etc/kubernetes/certs/ca.crt
```

**作用：** 指定用于验证连接到 API Server 的客户端证书的 CA 证书。

**说明：** 当客户端（如 kubelet、scheduler）使用证书连接时，API Server 会用此 CA 验证其合法性。

---

### 3. ServiceAccount 相关配置

**参数：**
```bash
--service-account-key-file=/etc/kubernetes/certs/sign_sa_pub.key
```

**作用：** 指定用于验证 ServiceAccount Token 的公钥（通常是 RSA 或 ECDSA 公钥）。

**注意：** 在较新版本中，如果使用 `--service-account-signing-key-file`，此字段可能被忽略或用于兼容。

---

**参数：**
```bash
--service-account-signing-key-file=/etc/kubernetes/certs/sign_sa_pri.key
```

**作用：** 指定用于签发 ServiceAccount Token 的私钥（JWT 签名密钥）。

---

**参数：**
```bash
--service-account-issuer=https://kubernetes.default.svc.cluster.local
```

**作用：** 设置 ServiceAccount Token 的 iss（issuer）声明值。

**说明：** 这是 OIDC 兼容的一部分，用于外部系统验证 Token 来源。

> ✅ 这三项共同启用了 Bound Service Account Token Volume（Kubernetes v1.22+ 默认启用），替代旧的 Secret-based Token。

---

### 4. etcd 连接安全配置

**参数：**
```bash
--etcd-cafile=/etc/kubernetes/certs/etcd/Etcdtrustca.crt
--etcd-certfile=/etc/kubernetes/certs/etcd/etcd-client.crt
--etcd-keyfile=/etc/kubernetes/certs/etcd/etcd-client.key
```

**作用：** API Server 与 etcd 通信时使用的 TLS 客户端证书、私钥和 CA。

**说明：** 确保 API Server 能安全地连接到启用了 mTLS 的 etcd 集群。

---

### 5. 聚合层（Aggregation Layer）/ 扩展 API（如 metrics-server）的安全配置

**参数：**
```bash
--requestheader-client-ca-file=/etc/kubernetes/certs/proxy-ca.crt
--proxy-client-cert-file=/etc/kubernetes/certs/proxy.crt
--proxy-client-key-file=/etc/kubernetes/certs/proxy.key
```

**作用：** 用于验证来自聚合 API 服务器（如 metrics-server、custom metrics adapters）的请求。

**机制：**
- 前端代理（如 kube-aggregator）使用 `proxy.crt/key` 向 API Server 发起请求。
- API Server 用 `proxy-ca.crt` 验证该代理的身份。
- 同时，代理会在请求头中携带原始用户信息（如 `X-Remote-User`），API Server 信任这些头（因为已验证代理身份）。

**关键用途：** 支持 API Aggregation 和 `kubectl auth proxy` 等高级功能。

> ⚠️ 这些参数配置不当可能导致权限提升漏洞，务必严格限制 `proxy-ca.crt` 只签发给可信组件。

---

### 6. API Server 与 Kubelet 通信的安全配置

**参数：**
```bash
--kubelet-certificate-authority=/etc/kubernetes/certs/apiserver_to_kubelet_cert/ca.crt
--kubelet-client-certificate=/etc/kubernetes/certs/apiserver_to_kubelet_cert/client.crt
--kubelet-client-key=/etc/kubernetes/certs/apiserver_to_kubelet_cert/client.key
```

**作用：** API Server 主动连接 Kubelet（例如执行 `kubectl logs`、`exec`、`port-forward`）时使用的客户端证书和验证 Kubelet 服务端证书的 CA。

**说明：**
- `kubelet-client-*`：API Server 作为客户端向 Kubelet 证明自己身份。
- `kubelet-certificate-authority`：用于验证 Kubelet 提供的 TLS 证书（防止中间人攻击）。

> 🔒 如果未配置，API Server 可能会跳过 Kubelet 证书验证（不安全），或无法执行某些操作。

---

## 安全机制总结

| 功能 | 安全机制 |
|------|----------|
| API Server 对外服务 | TLS + 客户端证书认证 |
| 与 etcd 通信 | mTLS（双向 TLS） |
| 与 Kubelet 通信 | mTLS（API Server 作为客户端） |
| ServiceAccount Token | JWT 签名与验证（OIDC 兼容） |
| 扩展 API（聚合层） | 受信任的前端代理 + 请求头传递身份 |

  
  

  

  


  
  
  
  
 
