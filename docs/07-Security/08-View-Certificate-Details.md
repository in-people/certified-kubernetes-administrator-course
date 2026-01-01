# View Certificate Details
  - Take me to [Video Tutorial](https://kodekloud.com/topic/view-certificate-details/)
  
In this section, we will take a look how to view certificates in a kubernetes cluster. 如何在 Kubernetes 集群中查看证书   

## View Certs 
"The Hard Way"（手动部署方式）  
命令：cat /etc/systemd/system/kube-apiserver.service  
含义：这个服务的配置文件里，会包含启动 kube-apiserver 时指定的命令行参数，其中 --tls-cert-file 和 --tls-private-key-file 等参数就指明了证书和密钥的路径。  

“kubeadm”（工具化部署方式）   
命令：cat /etc/kubernetes/manifests/kube-apiserver.yaml    
含义：这个 YAML 文件定义了 kube-apiserver 静态 Pod。在这个文件的 spec.containers[].command 部分，你可以找到启动参数，同样可以找到证书和密钥的路径（例如 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt）。  

 ![hrd](../../images/hrd.PNG)


API Server 自身 TLS 证书：
- 配置参数：
  - `--tls-cert-file=/etc/kubernetes/pki/apiserver.crt` (服务器证书)
  - `--tls-private-key-file=/etc/kubernetes/pki/apiserver.key` (服务器私钥)
- 作用：用于 HTTPS 服务（端口 6443），让客户端（如 kubectl）能安全地连接 API Server。

客户端 CA 证书：
- 配置参数：
  - `--client-ca-file=/etc/kubernetes/pki/ca.crt`
- 作用：用于验证客户端证书（如 kubectl、kubelet 等持有的证书）的颁发者。这是 Kubernetes 集群的根证书。

连接 Etcd 的证书：
- 配置参数：
  - `--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt` (Etcd 的 CA 证书)
  - `--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt` (API Server 作为 Etcd 客户端的证书)
  - `--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key` (对应私钥)
- 作用：让 API Server 能通过 TLS 双向认证安全地访问 Etcd 集群。

连接 Kubelet 的证书：
- 配置参数：
  - `--kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt`
  - `--kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key`
- 作用：让 API Server 能通过 TLS 认证安全地连接到各个节点上的 Kubelet。

前端代理客户端证书：
- 配置参数：
  - `--proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt`
  - `--proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key`
- 作用：用于聚合层（Aggregation Layer）的扩展 API 服务器认证。

 ![hrd1](../../images/hrd1.PNG)
 
 - To view the details of the certificate
   ```
   $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
   ```
### 证书主体信息
- **Subject（主题）**：CN=kube-apiserver
  - 表示这个证书是颁发给 kube-apiserver 这个通用名称的。
- **Issuer（颁发者）**：CN=kubernetes
  - 这个证书是由集群自己的 CA（证书颁发机构，名称为 kubernetes）签发的。这符合自签名（self-signed）或私有 PKI 的典型特征。

### 证书身份信息说明
这张图展示了 kube-apiserver 证书的完整"身份信息"。它说明了：

- **谁签发的**：集群自己的 CA (kubernetes)。
- **颁发给谁**：kube-apiserver。
- **能用在哪里**：
  - **目的**：作为 TLS 服务器。
  - **可用的地址**：列出了所有集群内可能用来访问 API Server 的 DNS 名称和 IP 地址（master, kubernetes, 10.96.0.1 等）。
  - **有效期限**：展示了证书的生命周期（此例中已过期，可作为历史参考或警示）。
  
   ![hrd2](../../images/hrd2.PNG)
   
#### Follow the same procedure to identify information about of all the other certificates

##### Kubernetes 核心证书信息汇总

这是一个关于 Kubernetes（使用 kubeadm 部署）核心证书信息的汇总表格。它整理了集群中关键 TLS 证书的路径、用途和属性，用于快速查阅和管理。

##### 表格内容解读
表格列出了 /etc/kubernetes/pki/ 目录下最重要的几对证书和密钥文件，并说明了它们的身份信息。

##### 各列含义
- **Certificate Path**：证书或密钥在主机上的文件路径。

- **CN Name (Common Name)**：证书的"通用名称"，标识了证书持有者的身份。

- **ALT Names (Subject Alternative Names)**：证书的主体备用名称，通常是 DNS 名称或 IP 地址，用于扩展身份验证。

- **Organization**：证书所属的组织（O字段）。在 Kubernetes 证书中，这常用来表示用户组。

- **Issuer**：证书的颁发者。对于客户端证书，通常是集群的根 CA (kubernetes)；对于根证书，可能是自签 (self)。

- **Expiration**：证书的过期时间。

   ![hrd3](../../images/hrd3.PNG)
   
## Inspect Server Logs - Hardware setup
- Inspect server logs using journalctl
### Etcd 服务日志分析

这是查看 Etcd 服务日志的内容截图。它展示了如何使用 journalctl 命令检查 Kubernetes 集群中 Etcd 服务的启动和运行状态。

#### 核心内容解读
1. **使用的命令**
   ```
   $ journalctl -u etcd.service -l
   ```
   - `journalctl`：Linux 系统上查看系统日志（journal）的命令。
   - `-u etcd.service`：-u 参数指定要查看的系统服务单元。这里查看的是 etcd.service。
   - `-l`：--full 的缩写，表示显示完整的日志条目（不截断长行）。
   
   这个命令是排查 Etcd 相关问题的首要步骤。

2. **客户端通信 TLS 配置（最重要的一行）**：
   ```
   embed: ClientTLS: cert = /etc/kubernetes/pki/etcd/server.crt, key = /etc/kubernetes/pki/etcd/server.key, ca = , trusted-ca = /etc/kubernetes/pki/etcd/ca.crt, client-cert-auth=true
   ```
   - `server.crt/server.key`：这是 Etcd 服务器的证书和私钥，用于向连接的客户端（如 kube-apiserver）证明自己的身份。
   - `ca.crt`：这是 Etcd 信任的 CA 证书。客户端（如 kube-apiserver）必须持有由此 CA 签发的客户端证书，连接才会被接受。
   - `client-cert-auth=true`：启用了客户端证书认证。这意味着 Etcd 要求所有客户端连接都必须提供有效的、由 trusted-ca（即 ca.crt）签发的客户端证书。这是 Kubernetes 安全性的核心。

  ```
  $ journalctl -u etcd.service -l
  ```
  
  ![hrd4](../../images/hrd4.PNG)
  
## Inspect Server Logs - kubeadm setup
- View logs using kubectl
  ```
  $ kubectl logs etcd-master
  ```
  ![hrd5](../../images/hrd5.PNG)
  
- View logs using docker ps and docker logs
  ```
  $ docker ps -a
  $ docker logs <container-id>
  ```
  ![hrd6](../../images/hrd6.PNG)
  
#### K8s Reference Docs
- https://kubernetes.io/docs/setup/best-practices/certificates/#certificate-paths
  
  

  

   
   

