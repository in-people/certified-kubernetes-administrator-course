# Certificate API
  - Take me to [Video Tutorial](https://kodekloud.com/topic/certificates-api/)
  
In this section, we will take a look at how to manage certificates and certificate API's in kubernetes

## CA (Certificate Authority) 证书颁发机构
- The CA is really just the pair of key and certificate files that we have generated, whoever gains access to these pair of files can sign any certificate for the kubernetes environment.
- CA 本质上就是一对密钥和证书文件
- 任何获得这对文件的人都可以为 Kubernetes 环境签署任何证书
- 这是 Kubernetes 集群安全的基础

### TLS 工作原理

Kubernetes 提供了一个内置的证书 API，用于管理证书签发过程：

### 用户发起请求：

- 用户生成私钥
- 创建证书签名请求（CSR）
- 将 CSR 提交给 Kubernetes

### 管理员处理：

- 管理员创建 CertificateSigningRequest 对象
- 审核并批准请求
- Kubernetes 自动签发证书
  
#### Kubernetes has a built-in certificates API that can do this for you. 
- With the certificate API, we now send a certificate signing request (CSR) directly to kubernetes through an API call.
   
  ![csr](../../images/csr.PNG)
   
#### This certificate can then be extracted and shared with the user.
- A user first creates a key 生成私钥 
  ```
  $ openssl genrsa -out jane.key 2048
  ```
- Generates a CSR 创建 CSR（证书签名请求） 
  ```
  $ openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
  ```
- Sends the request to the administrator and the adminsitrator takes the key and creates a CSR object, with kind as "CertificateSigningRequest" and a encoded "jane.csr"
  ```
  apiVersion: certificates.k8s.io/v1beta1
  kind: CertificateSigningRequest
  metadata:
    name: jane
  spec:
    groups:
    - system:authenticated
    usages:
    - digital signature
    - key encipherment
    - server auth
    request:
      <certificate-goes-here>
  ```
  $ cat jane.csr |base64 
  $ kubectl create -f jane.yaml
  ```
 ![csr1](../../images/csr1.PNG)
  
- To list the csr's  # 查看所有 CSR
  ```
  $ kubectl get csr
  ```
- Approve the request # 批准证书请求
  ```
  $ kubectl certificate approve jane
  ```
- To view the certificate # 查看证书详情
  ```
  $ kubectl get csr jane -o yaml
  ```
- To decode it
  ```
  $ echo "<certificate>" |base64 --decode
  ```
  
  ![csr2](../../images/csr2.PNG)
  
#### All the certificate releated operations are carried out by the controller manager. 
- If anyone has to sign the certificates they need the CA Servers, root certificate and private key. The controller manager configuration has two options where you can specify this.
证书相关的所有操作都由控制器管理器（Controller Manager）处理。控制器管理器需要以下配置来签署证书：  
CA 证书路径：--client-ca-file  
CA 私钥路径：--cluster-signing-cert-file 和 --cluster-signing-key-file  
  ![csr3](../../images/csr3.PNG)
  
  ![csr4](../../images/csr4.PNG)
  
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
 
  


