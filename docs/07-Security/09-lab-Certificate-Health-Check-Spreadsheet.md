# lab Certificate Health-Check Spreadsheet
  - Take me to [Spreadsheet](https://kodekloud.com/topic/certificate-health-check-spreadsheet/)
  
# Kubernetes TLS 证书配置与故障排除

## Kube API Server 证书配置

### 识别 Kube API 服务器使用的证书文件
```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep tls-cert
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
```
**中文翻译**：识别 Kube API 服务器使用的证书文件

### 识别用于向 ETCD 服务器认证 Kube-apiserver 的证书文件
```bash
- --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
```
**中文翻译**：识别用于向 ETCD 服务器认证 Kube-apiserver 的证书文件

### 识别用于向 Kubelet 服务器认证 Kubeapi-server 的密钥
```bash
- --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.key
```
**中文翻译**：识别用于向 Kubelet 服务器认证 Kubeapi-server 的密钥

### Kube API Server 配置文件中的证书相关参数
```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep crt
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
```

## ETCD 服务器证书配置

### 识别用于托管 ETCD 服务器的 ETCD 服务器证书
```bash
cat /etc/kubernetes/manifests/etcd.yaml | grep cert
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
      name: etcd-certs
    name: etcd-certs
```
**中文翻译**：识别用于托管 ETCD 服务器的 ETCD 服务器证书

### 识别用于服务 ETCD 服务器的 ETCD 服务器 CA 根证书
- **配置参数**：`--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt`
- **说明**：ETCD 可以拥有自己的 CA。因此，这可能与 kube-api 服务器使用的 CA 证书不同。

```bash
cat /etc/kubernetes/manifests/etcd.yaml | grep trusted-ca
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```
**中文翻译**：识别用于服务 ETCD 服务器的 ETCD 服务器 CA 根证书。ETCD 可以拥有自己的 CA。因此，这可能与 kube-api 服务器使用的 CA 证书不同。

## 证书详细信息

### Kube API 服务器证书的通用名称 (CN)
```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
```

输出示例：
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 2842320679175761062 (0x2771f3993404aca6)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Jan  1 06:40:43 2026 GMT
            Not After : Jan  1 06:45:43 2027 GMT
        Subject: CN = kube-apiserver
        Subject Public Key Info:
```
**中文翻译**：Kube API 服务器证书上配置的通用名称 (CN) 是什么？

### Kube API 服务器证书的备用名称
```
X509v3 Subject Alternative Name: 
    DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:172.20.0.1, IP Address:192.168.142.220
```
**中文翻译**：以下哪个备用名称没有配置在 Kube API 服务器证书上？

### ETCD 服务器证书的通用名称 (CN)
```bash
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text
```

输出示例：
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 8340057676506871010 (0x73bdd644ee7634e2)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = etcd-ca
        Validity
            Not Before: Jan  1 06:40:43 2026 GMT
            Not After : Jan  1 06:45:43 2027 GMT
        Subject: CN = controlplane
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```
**中文翻译**：ETCD 服务器证书上配置的通用名称 (CN) 是什么？

### 根 CA 证书有效期
```bash
openssl x509 -in /etc/kubernetes/pki/ca.crt -text
```

输出示例：
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1872462352206841682 (0x19fc5296ae94c752)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Jan  1 06:40:43 2026 GMT
            Not After : Dec 30 06:45:43 2035 GMT
        Subject: CN = kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```
**中文翻译**：根 CA 证书从签发日期起有效期多长？

## Kube API 服务器故障排除

### 问题描述
Kube API 服务器再次停止！检查它。检查 Kube API 服务器日志并识别根本原因并修复问题。

### 排查步骤
1. 运行以下命令识别 kube-api 服务器容器：
   ```bash
   crictl ps -a | grep kube-apiserver
   ```

2. 运行以下命令查看日志：
   ```bash
   crictl logs container-id
   ```

### 问题分析
如果检查控制平面的 kube-apiserver 容器，可以看到它频繁退出。

```bash
root@controlplane:~# crictl ps -a | grep kube-apiserver
1fb242055cff8       529072250ccc6       About a minute ago   Exited              kube-apiserver            3                   ed2174865a416       kube-apiserver-controlplane
```

检查此退出容器的日志，会看到以下错误：
```bash
root@controlplane:~# crictl logs --tail=2 1fb242055cff8  
W0916 14:19:44.771920       1 clientconn.go:1331] [core] grpc: addrConn.createTransport failed to connect to {127.0.0.1:2379 127.0.0.1 <nil> 0 <nil>}. Err: connection error: desc = "transport: authentication handshake failed: x509: certificate signed by unknown authority". Reconnecting...
E0916 14:19:48.689303       1 run.go:74] "command failed" err="context deadline exceeded"
```

**错误分析**：这表明 kube-apiserver 使用的 ETCD CA 证书存在问题。将其更正为使用文件 `/etc/kubernetes/pki/etcd/ca.crt`。

### 解决方案
一旦 YAML 文件保存后，等待 kube-apiserver pod 就绪。这可能需要几分钟时间。
