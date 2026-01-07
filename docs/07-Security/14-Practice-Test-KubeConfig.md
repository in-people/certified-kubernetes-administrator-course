# Kubernetes KubeConfig 实践测试

## 实践测试说明
- 了解 Kubernetes KubeConfig 文件的配置和使用
- 参考练习测试：[Practice Test](https://kodekloud.com/topic/practice-test-kubeconfig/)

## 问题描述与解决方案

### 1. 查找 kubeconfig 文件
- 查找位于 `/root/.kube` 目录下的 kube 配置文件

<details>
```
$ ls -l /root/.kube
```
</details>

### 2. 查看集群配置信息
- 执行命令 `kubectl config view` 并统计集群数量

<details>
```
$ kubectl config view
```
</details>

**默认配置示例：**
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

### 3. 统计用户和上下文数量
- 执行命令 `kubectl config view` 并统计用户数量

<details>
```
$ kubectl config view
```
</details>

- 默认 kubeconfig 文件中定义了多少个上下文？

<details>
```
$ kubectl config view
```
</details>

- 执行命令 `kubectl config view` 并查找用户名

<details>
```
$ kubectl config view
```
</details>

- 默认 kubeconfig 文件中配置的集群名称是什么？

<details>
```
$ kubectl config view
```
</details>

### 4. 自定义 kubeconfig 文件操作

#### 查看 my-kube-config 文件
- 执行命令 `kubectl config view --kubeconfig my-kube-config`

<details>
```
$ kubectl config view --kubeconfig my-kube-config
```
</details>

- my-kube-config 文件中配置了多少个上下文？

<details>
```
$ kubectl config view --kubeconfig my-kube-config
```
</details>

- 'research' 上下文中配置了哪个用户？

<details>
```
$ kubectl config view --kubeconfig my-kube-config
```
</details>

- 'aws-user' 配置的客户端证书文件名是什么？

<details>
```
$ kubectl config view --kubeconfig my-kube-config
```
</details>

- my-kube-config 文件中当前设置的上下文是什么？

```
kubectl config current-context --kubeconfig my-kube-config
test-user@development
```

<details>
```
$ kubectl config view --kubeconfig my-kube-config
```
</details>

### 5. 切换上下文操作
- 执行命令**切换到 research 上下文**

<details>
```
$ kubectl config --kubeconfig=/root/my-kube-config use-context research
```
</details>

### 6. 替换默认 kubeconfig 文件
- 将默认 kubeconfig 文件的内容替换为 my-kube-config 文件的内容

<details>
```
$ mv .kube/config .kube/config.bak
$ cp /root/my-kube-config .kube/config
```
</details>

### 7. 修复证书路径错误
- kubeconfig 文件中的证书路径不正确，需要修复。所有用户证书都存储在 `/etc/kubernetes/pki/users` 目录下

<details>
```
$ kubectl get pods
master $ ls
dev-user.crt  dev-user.csr  dev-user.key
master $ vi /root/.kube/config
master $ grep dev-user.crt /root/.kube/config
  client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt
master $ pwd
/etc/kubernetes/pki/users/dev-user
master $ kubectl get pods
No resources found in default namespace.
</details>

### 8. 使用特定用户访问集群
- 想要使用 dev-user 来访问 test-cluster-1。设置当前上下文到正确的配置，以便可以这样做。
- 一旦确定了正确的上下文，使用 `kubectl config use-context` 命令。

**命令示例：**
- 要使用该上下文，运行命令：`kubectl config --kubeconfig=/root/my-kube-config use-context research`
- 要知道当前上下文，运行命令：`kubectl config --kubeconfig=/root/my-kube-config current-context`

**执行结果：**
```
controlplane ~ ➜  kubectl config --kubeconfig=/root/my-kube-config use-context research
Switched to context "research".

controlplane ~ ➜  kubectl config --kubeconfig=/root/my-kube-config current-context
research
```

### 9. 设置默认 kubeconfig 文件
- 我们不想在每个 kubectl 命令中都指定 kubeconfig 文件选项。
- 将 my-kube-config 文件设置为默认的 kubeconfig 文件，并使其在所有会话中保持持久性，而不覆盖现有的 ~/.kube/config 文件。确保任何配置更改在重启和新的 shell 会话中都保持持久。

**注意事项：** 不要忘记在现有会话中加载（source）配置文件以使其生效。例如：
```
source ~/.bashrc
```

**操作步骤：**

1. 将 my-kube-config 文件添加到 KUBECONFIG 环境变量：
2. 打开 shell 配置文件：
```
vi ~/.bashrc
```
3. 添加以下行之一来导出变量：
```
export KUBECONFIG=/root/my-kube-config
# 或者
export KUBECONFIG=~/my-kube-config
# 或者
export KUBECONFIG=$HOME/my-kube-config
```
4. 应用更改：

重新加载 shell 配置以在当前会话中应用更改：
```
source ~/.bashrc
```

## my-kube-config.yaml 配置文件示例

```yaml
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: kubernetes-on-aws
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: test-cluster-1
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
preferences: {}
```
