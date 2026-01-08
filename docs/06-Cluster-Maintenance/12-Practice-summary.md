# Kubernetes 常用命令与笔记

## 一、节点维护 - kubectl drain

### 基本用法

```bash
kubectl drain node01 --ignore-daemonsets
```

### 命令作用

将 node01 节点设置为不可调度，并驱逐（删除）其上的所有 Pod

### 各部分说明

- `kubectl drain node01` - 准备排空节点，会：
  - 将节点标记为 `unschedulable`（阻止新 Pod 调度到此节点）
  - 驱逐该节点上所有非 DaemonSet/静态 Pod
  - 确保其他节点有足够资源接收被驱逐的 Pod

- `--ignore-daemonsets` - 忽略 DaemonSet 管理的 Pod
  - 默认情况下，如果节点上有 DaemonSet Pod，`drain` 会失败
  - 加上此参数后，会保留 DaemonSet Pod 不驱逐（因为它们需要在每个节点上运行）

### 典型使用场景

- 节点维护前（如升级内核、更换硬件）
- 节点下线前
- 滚动更新集群节点

### 恢复节点调度

```bash
# 维护完成后，恢复节点调度
kubectl uncordon node01

# 查看 drain 结果
kubectl describe node node01
```

---

## 二、drain 命令失败处理

### 问题：为什么第二次 drain 失败？

**原因**：节点上有一个不属于 ReplicaSet 的 Pod（独立 Pod）

**解决方法**：使用 `--force` 参数强制删除

```bash
kubectl drain node01 --ignore-daemonsets --force
```

### 输出示例

```
node/node01 already cordoned
Warning: deleting Pods that declare no controller: default/hr-app; ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-m249v, kube-system/kube-proxy-l5vk6
evicting pod default/hr-app
```

### 检查 Pod 状态

```bash
kubectl get pods -o wide
```

---

## 三、节点污点（Taints）检查

### 检查节点是否可调度

```bash
# 检查 controlplane 节点
kubectl describe node controlplane | grep -i taints

# 检查 node01 节点
kubectl describe node node01 | grep -i taints
```

### 输出说明

- `Taints: <none>` - 节点没有污点，可以调度普通 Pod
- 有污点的节点只能调度特定的 Pod（通过容忍度 Toleration）

---

## 四、集群升级策略

### 升级策略

**要求**：用户访问应用不能受影响，无法配置新虚拟机

**方案**：逐个节点升级，同时将工作负载迁移到其他节点（滚动升级）

---

## 五、kubeadm 集群升级

### 检查可升级版本

```bash
kubeadm upgrade plan
```

### 升级计划输出解读

#### 当前版本信息
```
[upgrade/versions] Cluster version: 1.33.0
[upgrade/versions] kubeadm version: v1.33.0
[upgrade/versions] Target version: v1.33.7
[upgrade/versions] Latest version in the v1.33 series: v1.33.7
```

#### 需要手动升级的组件
```
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.33.0   v1.33.7
kubelet     node01         v1.33.0   v1.33.7
```

#### 自动升级的组件
```
COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.33.0    v1.33.7
kube-controller-manager   controlplane   v1.33.0    v1.33.7
kube-scheduler            controlplane   v1.33.0    v1.33.7
kube-proxy                               1.33.0     1.33.7
CoreDNS                                  v1.10.1    v1.12.0
etcd                      controlplane   3.5.21-0   3.5.21-0
```

### 执行升级

```bash
# 1. 先升级 kubeadm 到目标版本
apt-get update && apt-get install -y kubeadm=1.33.7-00

# 2. 应用升级
kubeadm upgrade apply v1.33.7

# 3. 升级 kubelet
apt-get install -y kubelet=1.33.7-00

# 4. 重启 kubelet
systemctl restart kubelet
```

### 升级特定版本

```bash
# 升级到精确版本 v1.34.0
kubeadm upgrade apply v1.34.0
```

---

## 六、ETCD 备份与恢复

### 查看 ETCD 版本

```bash
# 通过 Pod 查看
kubectl describe pod etcd-controlplane -n kube-system | grep Image

# 输出示例
image: registry.k8s.io/etcd:3.6.4-0
```

### ETCD 配置信息

#### 客户端监听地址
```
--listen-client-urls=https://127.0.0.1:2379,https://192.168.183.196:2379
```

#### 证书文件位置
- 服务器证书：`/etc/kubernetes/pki/etcd/server.crt`
- CA 证书：`/etc/kubernetes/pki/etcd/ca.crt`
- 私钥：`/etc/kubernetes/pki/etcd/server.key`

### ETCD 快照备份

```bash
etcdctl --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/snapshot-pre-boot.db
```

### 使用场景

- 节点维护前的备份
- 升级前的快照
- 灾难恢复准备

---

## 七、常用命令速查

```bash
# 节点维护
kubectl drain <node> --ignore-daemonsets --force
kubectl uncordon <node>
kubectl cordon <node>  # 仅标记为不可调度，不驱逐 Pod

# 查看节点状态
kubectl get nodes
kubectl describe node <node>
kubectl top node  # 查看资源使用

# 查看 Pod 状态
kubectl get pods -o wide
kubectl describe pod <pod-name>

# 升级相关
kubeadm upgrade plan
kubeadm upgrade apply <version>

# ETCD 备份
etcdctl snapshot save <file-path> --endpoints=<url> --cacert=<cert> --cert=<cert> --key=<key>
```
