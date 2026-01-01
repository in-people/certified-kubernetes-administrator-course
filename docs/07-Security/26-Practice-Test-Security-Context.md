# Kubernetes 安全上下文（Security Context）实践测试

## 实践测试说明
- 了解 Kubernetes 安全上下文的配置和使用
- 参考练习测试：[Practice Test](https://kodekloud.com/topic/practice-test-security-contexts/)

## 问题描述与解决方案

### 1. 查看 Pod 中的用户身份
- 执行命令 `kubectl exec ubuntu-sleeper -- whoami` 并统计 Pod 数量

<details>
```
$ kubectl exec ubuntu-sleeper -- whoami
```
```
controlplane ~ ✖ kubectl exec -it  ubuntu-sleeper -- /bin/sh
whoami
root
```
</details>

### 2. 设置安全上下文以特定用户运行
- 设置安全上下文以用户 ID 1010 运行

<details>
```
$ kubectl get pods ubuntu-sleeper -o yaml > ubuntu.yaml
$ kubectl delete pod ubuntu-sleeper
$ vi ubuntu.yaml (添加 securityContext 部分)
  securityContext:
    runAsUser: 1010
$ kubectl create -f ubuntu.yaml
```
</details>

### 3. 安全上下文规则说明
- 容器安全上下文（securityContext）中定义的用户 ID 会覆盖 Pod 中的用户 ID
- Pod 安全上下文（securityContext）中定义的用户 ID 会应用到容器中的所有 Pod

### 4. 修改系统时间测试
- 执行命令尝试修改系统时间：`kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'`

<details>
```
$ kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
```
</details>

### 5. 为容器添加系统权限
- 向容器的安全上下文（securityContext）添加 SYS_TIME 权限

<details>
```
$ kubectl get pods ubuntu-sleeper -o yaml > ubuntu.yaml
$ kubectl delete pod ubuntu-sleeper
$ vi ubuntu.yaml

在容器部分添加以下内容：

securityContext:
    capabilities:
      add: ["SYS_TIME"]
      
$ kubectl create -f ubuntu.yaml
```

---
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:      # 更新 securityContext
      capabilities:
        add: ["SYS_TIME"]
```
</details>

### 6. 验证权限设置
- 现在尝试在 Pod 中运行以下命令来设置日期。如果安全权限添加正确，则应该能够成功。如果失败，请确保将用户更改回 root。

<details>
```
$ kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
```
</details>
