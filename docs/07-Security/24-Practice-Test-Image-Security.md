# Kubernetes 镜像安全实践测试

## 实践测试说明
- 了解 Kubernetes 镜像安全配置
- 参考练习测试：[Practice Test](https://kodekloud.com/topic/practice-test-image-security/)

## 核心概念

### Docker Registry Secret 类型
- 问题：什么 secret 类型必须用于 Docker Registry？
- 答案：`docker-registry`
- 说明：在 Kubernetes 中，为了从私有 Docker Registry 拉取镜像，你需要创建 docker-registry 类型的 Secret。

## 实践场景

### 1. 应用镜像更新
我们决定使用来自内部私有仓库的应用程序的修改版本。请将部署的镜像更新为使用来自 myprivateregistry.com:5000 的新镜像。

```bash
root@controlplane ~ ➜   kubectl get deployment web -oyaml | grep image
      - image: myprivateregistry.com:5000/nginx:alpine
```

### 2. 创建私有仓库凭证 Secret
创建一个 Secret 对象，其中包含访问镜像仓库所需的凭证：

- **名称**: private-reg-cred
- **用户名**: dock_user
- **密码**: dock_password
- **服务器**: myprivateregistry.com:5000
- **邮箱**: dock_user@myprivateregistry.com

**创建命令：**
```bash
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
```

### 3. 配置部署使用私有仓库
配置部署以使用新 Secret 中的凭证从私有仓库拉取镜像：

```yaml
imagePullSecrets:
  - name: private-reg-cred
```

## 操作步骤

### 1. 查看当前应用镜像
- 检查应用程序正在使用的镜像是什么？

<details>
```
$ kubectl get deploy -o wide
```
</details>

### 2. 更新部署镜像
- 使用 `kubectl edit deployment` 命令编辑镜像名称为 myprivateregistry.com:5000/nginx:alpine

<details>
```
$ kubectl edit deployment web
```
</details>

### 3. 检查 Pod 状态
- 执行命令 `kubectl get pods` 并检查 Pod 状态

<details>
```
$ kubectl get pods
```
</details>

### 4. 创建 Docker Registry Secret
- 执行命令创建 Secret

<details>
```
$ kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
```
</details>

### 5. 编辑部署添加镜像拉取凭证
- 使用 `kubectl edit deploy web` 命令编辑部署并添加 imagePullSecrets 部分，使用 private-reg-cred

<details>
```
$ kubectl edit deploy web
```
</details>

### 6. 验证部署状态
- 检查 Pod 状态，等待它们运行。现在你已成功配置部署从私有仓库拉取镜像

<details>
```
$ kubectl get pods
```
</details>
