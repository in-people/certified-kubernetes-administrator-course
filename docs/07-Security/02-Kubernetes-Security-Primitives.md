# Kubernetes Security Primitives
  - Take me to [Video Tutorial](https://kodekloud.com/topic/kubernetes-security-primitives/)
  
In this section, we will take a look at kubernetes security primitives

## Secure Hosts

 ![sech](../../images/sech.PNG)
  
## Secure Kubernetes
- We need to make two types of decisions.
  - Who can access?
  - What can they do?
 
  ![seck](../../images/seck.PNG)
  
## Authentication
- Who can access the API Server is defined by the Authentication mechanisms.
  
## Authorization
- Once they gain access to the cluster, what they can do is defined by authorization mechanisms.

## TLS Certificates
- All communication with the cluster, between the various components such as the ETCD Cluster, kube-controller-manager, scheduler, api server, as well as those running on the working nodes such as the kubelet and kubeproxy is secured using TLS encryption.

Kubernetes 集群内部通信的安全机制，核心要点如下：  
1. TLS（Transport Layer Security）加密  
TLS 是一种广泛使用的安全协议，用于在两个通信端点之间提供机密性（数据加密）、完整性（防篡改）和身份认证（通常通过证书）。  
在 Kubernetes 中，几乎所有关键组件之间的通信都强制使用 TLS，防止中间人攻击、窃听或伪造请求。  

2. 如何实现？  
每个组件都配置了：  
客户端/服务端 TLS 证书和私钥（用于身份认证和加密）  
受信任的 CA 证书（用于验证对方证书是否合法）  
这些证书通常由集群部署工具（如 kubeadm、kops、或手动 PKI）生成和分发。  

3. 为什么重要？  
如果没有 TLS，攻击者可能：  
窃取敏感数据（如 Secret、Pod 配置）  
伪装成 kubelet 删除 Pod  
篡改调度决策  
因此，启用并正确配置 TLS 是 Kubernetes 安全的基石。  

 ![tls](../../images/tls.PNG)
 
## Network Policies
What about communication between applications within the cluster?

  ![np](../../images/np.PNG)
  
