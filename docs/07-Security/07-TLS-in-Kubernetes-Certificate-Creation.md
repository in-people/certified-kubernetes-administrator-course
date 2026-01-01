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
   
   
   
  
  

  

  


  
  
  
  
 
