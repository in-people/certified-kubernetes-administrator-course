# Authentication
  - Take me to [Video Tutorial](https://kodekloud.com/topic/authentication/)
  
In this section, we will take a look at authentication in a kubernetes cluster

## Accounts

  ![auth1](../../images/auth1.PNG)
  
#### Different users that may be accessing the cluster security of end users who access the applications deployed on the cluster is managed by the applications themselves internally.

 ![acc1](../../images/acc1.PNG)
 
- So, we left with 2 types of users
  - Humans, such as the Administrators and Developers
  - Robots such as other processes/services or applications that require access to the cluster.
  

  ![acc2](../../images/acc2.PNG)
  
- All user access is managed by apiserver and all of the requests goes through apiserver.
 
  ![acc3](../../images/acc3.PNG)
  
## Authentication Mechanisms
- There are different authentication mechanisms that can be configured.

  ![auth2](../../images/auth2.PNG)
  
## Authentication Mechanisms - Basic
  
  ![auth3](../../images/auth3.PNG)
  
## kube-apiserver configuration
- If you set up via kubeadm then update kube-apiserver.yaml manifest file with the option.
  
  ![auth4](../../images/auth4.PNG)
  
## Authenticate User

- To authenticate using the basic credentials while accessing the API server specify the username and password in a curl command.
  ```
  $ curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
  ```
  ![auth5](../../images/auth5.PNG)
  
- We can have additional column in the user-details.csv file to assign users to specific groups.
-  -H "Authorization: Bearer ..."	认证头。这是向 Kubernetes API 证明身份的方式。 KpjCVbI7rCFAHYPkBzRb7gulcUc4B 是一个 Service Account Token（服务账户令牌）或 User Token（用户令牌）。它相当于访问集群的“密码” 。

  ![auth6](../../images/auth6.PNG)
  
## Note
 
 ![note](../../images/note.PNG)

# Kubernetes API 安全认证说明

## 1. kubectl 配置

查看 kubectl 配置：

```bash
kubectl config view --raw
```

配置内容：

```yaml
apiVersion: v1
clusters:
- cluster:
  certificate-authority: /etc/kubernetes/certs/ca.crt
  server: https://127.0.0.1:6443
  name: cluster.local
contexts:
- context:
  cluster: cluster.local
  user: kubectl
  name: kubectl-to-cluster.local
current-context: kubectl-to-cluster.local
kind: Config
users:
- name: kubectl
  user:
  client-certificate: /etc/kubernetes/certs/kubectl.crt
  client-key: /etc/kubernetes/certs/kubectl.key
```

## 2. 使用 curl 访问 Kubernetes API

```bash
curl --cacert /etc/kubernetes/certs/ca.crt \
     --cert /etc/kubernetes/certs/kubectl.crt \
     --key /etc/kubernetes/certs/kubectl.key \
     https://127.0.0.1:6443/api/v1/namespaces/default/services?limit=500
```

## 3. 认证机制说明

- **`--cacert`**: 验证服务器身份，确保连接到合法的 API Server

- **`--cert` 和 `--key`**: 让 curl 向 Kubernetes API Server 证明"我是谁"的身份凭证
  - 相当于登录时的用户名+密码
  - 这里采用基于证书的安全认证方式

### 权限说明

1. kubectl 自动使用这对证书/私钥向 API Server 证明："我是 kubectl 用户，属于 system:masters 组"

2. 如果证书中的 `O=system:masters`，Kubernetes 内置的 RBAC 规则会授予 `cluster-admin`（超级管理员）权限

3. 在 curl 中加上 `--cert` 和 `--key`，相当于让 curl 扮演 kubectl 的角色，拥有相同权限  
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/authentication/ 
  
  
