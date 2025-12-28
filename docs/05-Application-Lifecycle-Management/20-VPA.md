
https://cloud.tencent.com/developer/article/2327655    

# Key Differences: VPA vs HPA

| Feature | VPA (Vertical Scaling) | HPA (Horizontal Scaling) |
|--------|------------------------|--------------------------|
| **Scaling Method** | Increases CPU and memory of existing Pods | Adds/Removes Pods based on load |
| **Pod Behavior** | Restarts Pods to apply new resource values | Keeps existing Pods running |
| **Handles Traffic Spikes?** | ❌ No, because scaling requires a Pod restart | ✅ Yes, instantly adds more Pods |
| **Optimizes Costs?** | ✅ Prevents over-provisioning of CPU/memory | ✅ Avoids unnecessary idle Pods |
| **Best for** | Stateful workloads, CPU/memory-heavy apps (DBs, ML workloads) | Web apps, microservices, stateless services |
| **Example Use Cases** | Databases (MySQL, PostgreSQL), JVM-based apps, AI/ML workloads | Web servers (Nginx, API services), message queues, microservices |


# 实验目标

在本实验中，你将在 Kubernetes 集群中安装并配置 **Vertical Pod Autoscaler（VPA，垂直 Pod 自动扩缩容器）**。  
实验将引导你完成以下操作：

- 使用预定义的清单文件（manifests）安装 VPA；
- 克隆 VPA 官方代码仓库以实现更高级的控制；
- 部署一个示例应用，观察 VPA 如何与其交互；
- 学习如何通过 VPA 生成的日志排查问题。

完成本实验后，你将能够：

- 在 Kubernetes 集群中安装 VPA 及其核心组件（Recommender、Updater、Admission Controller）；
- 理解每个 VPA 组件的作用，以及它们如何协同实现高效的资源管理；
- 部署一个示例应用，并观察 VPA 如何为其推荐和调整 CPU/内存资源请求；
- 利用 VPA 组件（尤其是 Updater）生成的日志，排查与资源相关的应用问题。

> 本次动手实践将帮助你掌握在生产级 Kubernetes 环境中动态管理 Pod 资源的能力，确保应用程序始终以合适的资源请求高效运行。
安装vpa crd、rbac


# Clone the VPA Repository and Set Up the Vertical Pod Autoscaler

You are required to clone the Kubernetes Autoscaler repository into the /root directory and set up the Vertical Pod Autoscaler (VPA) by running the provided script.

## Steps:

### 1. Clone the repository:

First, navigate to the /root directory and clone the repository:

```
git clone https://github.com/kubernetes/autoscaler.git
```

### 2. Navigate to the Vertical Pod Autoscaler directory:

After cloning, move into the vertical-pod-autoscaler directory:

```
cd autoscaler/vertical-pod-autoscaler
```

### 3. Run the setup script:

Execute the provided script to deploy the Vertical Pod Autoscaler:

```
./hack/vpa-up.sh
```

By following these steps, the Vertical Pod Autoscaler will be installed and ready to manage pod resources in your Kubernetes cluster.  


# VPA更新器错误：副本集过少问题

您最近在Kubernetes集群中部署了一个Flask应用程序。然而，Vertical Pod Autoscaler (VPA) 的vpa-updater-XXXX Pod显示新部署的flask-app Pod可能存在一些问题。

检查vpa-updater-XXXX Pod的日志并观察以下消息：

检查vpa-updater-XXXX Pod的日志以识别flask-app部署的潜在问题。

当检查日志时，您会看到以下错误消息：

```
pods_eviction_restriction.go:226] **too few replicas** for **ReplicaSet** default/**flask-app-b6c9c4f78**
```

## 问题分析：

- Flask应用程序仅以1个副本Pod运行。
- Vertical Pod Autoscaler (VPA) 需要驱逐（移除）现有的Pod以使用更新的资源设置创建新的Pod。
- Kubernetes有一个安全功能，可以防止删除部署的最后一个Pod以避免服务停机。
- 当您只有1个副本且VPA尝试驱逐它时，Kubernetes会使用错误消息"too few replicas"阻止此操作。
- VPA想要优化您的Pod资源，但由于Kubernetes保护您的服务可用性而无法实现。
- 结果，VPA无法应用其资源建议，应用程序无法从自动资源优化中受益。

## 解决方案：

1. 增加副本数量：

```
kubectl scale deployment flask-app --replicas=2
```

2. 验证部署：

```
kubectl get deployment flask-app -o wide
```

确保DESIRED列显示更新的副本数量，并且CURRENT列与所需数量匹配。

3. 检查Pod状态：

```
kubectl get pods -l app=flask-app
```

等待所有Pod显示为Running状态。

您应该看到两个（或更多）Pod处于Running状态。

4. 验证VPA操作：

```
kubectl describe vpa flask-app
```

这将显示VPA的当前状态和它所做的任何建议。如果正常工作，您应该在输出中看到资源建议（CPU和内存）。

拥有2个副本后，Kubernetes可以安全地移除一个Pod，同时保持您的应用程序运行，使VPA能够正常工作。


```
kubectl describe vpa flask-app
```

这将显示VPA的当前状态和它所做的任何建议。如果正常工作，您应该在输出中看到资源建议（CPU和内存）。

拥有2个副本后，Kubernetes可以安全地移除一个Pod，同时保持您的应用程序运行，使VPA能够正常工作。

# VPA描述输出信息

以下是`kubectl describe vpa flask-app`命令的输出：

```
Name:         flask-app
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  autoscaling.k8s.io/v1
Kind:         VerticalPodAutoscaler
Metadata:
  Creation Timestamp:  2025-12-28T11:05:25Z
  Generation:          1
  Resource Version:    4805
  UID:                 edab4591-4f5a-425b-8a00-b4cd53c9e142
Spec:
  Resource Policy:
    Container Policies:
      Container Name:  *
      Controlled Resources:
        cpu
        memory
      Max Allowed:
        Cpu:     1
        Memory:  500Mi
      Min Allowed:
        Cpu:     100m
        Memory:  100Mi
  Target Ref:
    API Version:  apps/v1
    Kind:         Deployment
    Name:         flask-app
  Update Policy:
    Eviction Requirements:
      Change Requirement:  TargetHigherThanRequests
      Resources:
        cpu
        memory
    Update Mode:  Recreate
Status:
  Conditions:
    Last Transition Time:  2025-12-28T11:06:04Z
    Status:                True
    Type:                  RecommendationProvided
  Recommendation:
    Container Recommendations:
      Container Name:  flask-app
      Lower Bound:
        Cpu:     100m
        Memory:  250Mi
      Target:
        Cpu:     100m
        Memory:  250Mi
      Uncapped Target:
        Cpu:     25m
        Memory:  250Mi
      Upper Bound:
        Cpu:     100m
        Memory:  250Mi
Events:
  Type    Reason      Age   From         Message
  ----    ------      ----  ----         -------
  Normal  EvictedPod  104s  vpa-updater  VPA Updater evicted Pod flask-app-84d5df7d4c-jfn47 to apply resource recommendation.
```

以上输出显示VPA已经成功应用了资源推荐，Pod已经被驱逐以应用新的资源配置。

# VPA输出解释

以下是对`kubectl describe vpa flask-app`输出内容的详细解释：

## Metadata部分
- **Name**: flask-app - VPA资源的名称
- **Namespace**: default - VPA所在的命名空间
- **Creation Timestamp**: VPA资源创建的时间
- **UID**: VPA资源的唯一标识符

## Spec部分（规范配置）
- **Resource Policy**:
  - **Controlled Resources**: VPA监控和调整的资源类型，这里是CPU和内存
  - **Max Allowed**: 最大允许资源限制 - CPU为1核，内存为500Mi
  - **Min Allowed**: 最小允许资源限制 - CPU为100m，内存为100Mi
  - **Container Name**: * 表示适用于所有容器
- **Target Ref**: 指定VPA监控的目标资源，这里是名为flask-app的Deployment
- **Update Policy**: 更新策略设置为"Recreate"，表示VPA可以通过重新创建Pod来应用新的资源请求

## Status部分（状态信息）
- **Conditions**:
  - **RecommendationProvided**: 状态为True，表示VPA已经提供了资源建议
  - **Last Transition Time**: 状态变化的最后时间
- **Recommendation**: VPA提供的资源建议
  - **Container Name**: flask-app - 针对的容器名称
  - **Lower Bound**: 资源下限 - CPU 100m, 内存 250Mi
  - **Target**: 推荐的目标资源 - CPU 100m, 内存 250Mi
  - **Uncapped Target**: 无限制目标 - CPU 25m, 内存 250Mi
  - **Upper Bound**: 资源上限 - CPU 100m, 内存 250Mi

## Events部分（事件记录）
- **EvictedPod**: VPA成功驱逐了Pod `flask-app-84d5df7d4c-jfn47` 以应用资源建议，这是VPA正常工作的标志

总的来说，这个输出表明VPA正在正常工作，它已成功为flask-app部署提供了资源建议，并且已经成功驱逐并重新创建了Pod以应用新的资源配置。


针对您提到的VPA状态输出中各个字段的含义：

```
status:
  conditions:
  - lastTransitionTime: "2025-12-28T11:19:39Z"
    status: "True"
    type: RecommendationProvided
  recommendation:
    containerRecommendations:
    - containerName: flask-app-4
      lowerBound:
        cpu: 211m
      target:
        cpu: 410m
      uncappedTarget:
        cpu: 410m
      upperBound:
        cpu: "1"
```

## Status部分详解

### Conditions（状态条件）
- **type: RecommendationProvided** - 表示VPA是否已提供资源建议
  - **status: "True"** - 表示VPA已经成功提供了资源建议
  - **lastTransitionTime** - 状态最后一次变化的时间

### Recommendation（资源建议）
- **containerName** - 目标容器的名称

#### 各种资源边界说明
- **lowerBound（下限）** - CPU: 211m
  - 表示VPA计算出的容器所需的最低CPU资源
  - 这是基于容器历史使用情况的最低资源需求
  - 211m表示211毫核CPU（0.211核）

- **target（目标值）** - CPU: 410m
  - 这是VPA推荐的容器应使用的CPU资源
  - 这是VPA基于容器当前和历史使用情况计算出的理想资源值
  - 410m表示410毫核CPU（0.41核）

- **uncappedTarget（无限制目标）** - CPU: 410m
  - 这是在没有资源限制约束的情况下，VPA计算出的资源需求
  - 通常与target值相同，除非受到Max Allowed限制

- **upperBound（上限）** - CPU: "1"
  - 表示VPA计算出的容器所需的最大CPU资源
  - "1"表示1个CPU核心（1000m）
  - 这是容器在任何情况下都不应超过的资源上限

这些值表明VPA分析了flask-app-4容器的资源使用情况，建议将CPU资源设置为410m（0.41核），最低不应低于211m，最高不应超过1核。

