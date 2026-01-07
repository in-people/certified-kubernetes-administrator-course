# Practice Test - View Certificates
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-view-certificate-details/)
  
Solutions to practice test - view certificates
- Identify the certificate file used for the kube-api server
  识别kube-api server使用的证书  
  <details>
  ```bash
  $ cat /etc/kubernetes/manifest/kube-apiserver.yaml
  
  Answer: /etc/kubernetes/pki/apiserver.crt
  ```
  </details>
  
- Identify the Certificate file used to authenticate kube-apiserver as a client to ETCD Server
  识别用于 kube-apiserver 作为客户端向 ETCD 服务器进行身份验证的证书文件   

  <details>
  ```bash
  $ cat /etc/kubernetes/manifest/kube-apiserver.yaml
  Answer: /etc/kubernetes/pki/apiserver-etcd-client.crt
  ```
  </details>

- Look for kubelet-client-key option in the file /etc/kubernetes/manifests/kube-apiserver.yaml
  查找文件 /etc/kubernetes/manifests/kube-apiserver.yaml 中的 kubelet-client-key 选项
  <details>
  ```bash
  Answer: /etc/kubernetes/pki/apiserver-kubelet-client.key
  ```
  </details>
  
- Look for cert file option in the file /etc/kubernetes/manifests/etcd.yaml
  查找文件 /etc/kubernetes/manifests/etcd.yaml 中的证书文件选项
  <details>
  ```bash
  Answer: /etc/kubernetes/pki/etcd/server.crt
  ```
  </details>
  
- Look for CA Certificate in file /etc/kubernetes/manifests/etcd.yaml
  查找文件 /etc/kubernetes/manifests/etcd.yaml 中的CA证书
  <details>
  ```bash
  Answer: /etc/kubernetes/pki/etcd/ca.crt
  ```
  </details>
  
- Run the command openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  运行命令 openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  <details>
  ```bash
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  ```
  </details>
  
- Run the command openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text and look for issuer
  运行命令 openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text 并查找发行者
  <details>
  ```bash
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text 
  ```
  </details>
  
- Run the command openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text and look at Alternative Names
  运行命令 openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text 并查看备用名称
  <details>
  ```bash
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  ```
  </details>
  
- Run the command openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text and look for Subject CN.
  运行命令 openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text 并查找主题通用名称
  <details>
  ```bash
  $ openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text
  ```
  </details>
  
- Run the command openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text and check on the Expiry date.
  运行命令 openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text 并检查过期日期
  <details>
  ```bash
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  ```
  </details>
  
- Run the command 'openssl x509 -in /etc/kubernetes/pki/ca.crt -text' and look for validity
  运行命令 'openssl x509 -in /etc/kubernetes/pki/ca.crt -text' 并查找有效期
  <details>
  ```bash
  $ openssl x509 -in /etc/kubernetes/pki/ca.crt -text
  ```
  </details>
  
- Inspect the --cert-file option in the manifests file.
  检查清单文件中的 --cert-file 选项
  <details>
  ```bash
  $ vi /etc/kubernetes/manifests/etcd.yaml
  ```
  </details>
  
- ETCD has its own CA. The right CA must be used for the ETCD-CA file in /etc/kubernetes/manifests/kube-apiserver.yaml. 
  ETCD 有自己的CA。在 /etc/kubernetes/manifests/kube-apiserver.yaml 中必须为ETCD-CA文件使用正确的CA
  <details>
  ```bash
  View answer at /var/answers/kube-apiserver.yaml
  ```
  </details>
  
- Identify the ETCD Server CA Root Certificate used to serve ETCD Server.
  识别用于服务ETCD服务器的ETCD服务器CA根证书
  ETCD可以有自己的CA。所以这可能与kube-api服务器使用的CA证书不同。
  <details>
  ```bash
  Answer: /etc/kubernetes/pki/etcd/ca.crt
  ```
  </details>
  
- The kube-api server stopped again! Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue.
  kube-api服务器再次停止！检查一下。检查kube-api服务器日志，确定根本原因并解决问题。
  运行 crictl ps -a 命令来识别 kube-api 服务器容器。运行 crictl logs container-id 命令来查看日志。
  <details>
  ```bash
  # 解决步骤：
  crictl ps -a # 查找kube-api服务器容器
  crictl logs <container-id> # 查看容器日志
    
  # 修复方法：
  vi /etc/kubernetes/manifests/kube-apiserver.yaml
  # 修改成下面的一行
  - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
  
  # 错误日志：
  证书异常
  W0107 12:32:14.887553       1 logging.go:55] [core] [Channel #1 SubChannel #3]grpc: addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2379", ServerName: "127.0.0.1:2379", BalancerAttributes: {"<!%p(pickfirstleaf.managedByPickfirstKeyType={})>": "<!%p(bool=true)>" }}. Err: connection error: desc = "transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate signed by unknown authority"
F0107 12:32:17.858882       1 instance.go:232] Error creating leases: error creating storage factory: context deadline exceeded
  ```
  </details>
