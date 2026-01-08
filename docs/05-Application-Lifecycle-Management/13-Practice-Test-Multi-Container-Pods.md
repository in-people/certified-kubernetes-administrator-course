# Practice Test - Multi-Container Pods
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-multi-container-pods/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - name: lemon
    image: busybox
    command:
    - sleep
    - "1000"
  - name: gold
    image: redis
```


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  initContainers:
  - name: sidecar
    image: kodekloud/filebeat-configured
    restartPolicy: Always
    volumeMounts:
      - name: log-volume
        mountPath: /var/log/event-simulator/
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume
  volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp
      type: DirectoryOrCreate
```


这幅图展示了一个典型的 基于 Kubernetes 的日志收集与分析架构，使用了 Elastic Stack（Elasticsearch + Kibana）并结合 Filebeat 作为轻量级数据采集器。我们来逐部分解析它的含义：

## 🔍 图解说明

### 🟦 左侧：APP POD
- 表示一个运行在 Kubernetes 中的应用程序容器（Pod）。
- 这个应用会产生日志文件（如 app.log、error.log 等），这些日志需要被收集和分析。

### 🟧 虚线连接：Sidecar Filebeat
- Filebeat 是一个轻量级的开源数据采集器（属于 Elastic Beats 家族）。
- 它以 Sidecar 模式 部署在同一个 Pod 内，与主应用共享网络和存储空间。
- 作用是：
  - 实时监听应用的日志文件（例如 /var/log/app.log）。
  - 将新产生的日志内容读取并发送到 Elasticsearch。
- ✅ 使用 Sidecar 模式的好处：
  - 不影响主应用性能；
  - 可直接访问应用日志路径；
  - 易于部署和管理。

### 🟨 中间：ElasticSearch
- 接收来自 Filebeat 发送的日志数据。
- 将日志按索引（Index）结构化存储，并支持全文搜索、聚合分析等操作。
- 是整个系统的"数据大脑"。

### 🟩 右侧：Kibana
- 提供 Web 界面，用于可视化和探索 Elasticsearch 中的数据。
- 用户可以通过 Kibana：
  - 创建仪表盘（Dashboard）查看关键指标；
  - 查询日志（Discover 页面）；
  - 做趋势分析、错误排查、告警设置等。

### 👤 最右侧：用户
- 表示运维人员或开发人员通过 Kibana 查看日志、监控系统状态。

## 🔄 整体流程（从左到右）
1. App Pod 产生日志 → 写入本地文件。
2. Filebeat（Sidecar） 监听日志文件 → 实时抓取日志内容。
3. Filebeat 发送到 Elasticsearch → 日志被解析、索引、存储。
4. Kibana 连接 Elasticsearch → 用户通过图形界面查询、分析日志。

## ✅ 架构优势

| 优点 | 说明 |
|------|------|
| 轻量化 | Filebeat 很小（<10MB），资源占用低。 |
| 实时性高 | 日志几乎实时进入 ES，便于快速排查问题。 |
| 可扩展性强 | 支持多节点部署，适合生产环境。 |
| 易用性好 | Kibana 提供直观的 UI，无需写复杂 SQL 或代码即可分析。 |

## 💡 小贴士：实际部署建议
如果你的应用跑在 Kubernetes 上，推荐使用：
- DaemonSet 部署 Filebeat（每个节点运行一个实例）；
- 或者使用 Sidecar 模式（如图所示）；
- 使用 Elasticsearch Operator（如 ECK）简化集群管理；
- 对敏感日志启用加密传输（TLS）和身份认证。

## 🧩 总结一句话：
这张图描述的是：在 Kubernetes 中，通过 Sidecar 模式的 Filebeat 收集应用日志 → 存入 Elasticsearch → 用 Kibana 可视化分析，实现一套完整的可观测性（Observability）方案。

这是现代云原生系统中最常见的日志处理架构之一.

Solutions to practice test - multi-container pods
- Identify the number of containers running in the 'red' pod.
  
  <details>
  
  ```
  $ kubectl get pod red
  ```
  </details>
  
- Identify the name of the containers running in the 'blue' pod.

  <details>
  ```
  $ kubectl describe pod blue
  ```
  </details>
    
- Answer file is located at /var/answers/answer-yellow.yaml

  <details>
  ```
  $ kubectl create -f /var/answers/answer-yellow.yaml
  ```
  </details>
  
- We have deployed an application logging stack in the elastic-stack namespace. Inspect it.
  
  <details>
  ```
  $ kubectl get pods -n elastic-stack
  ```
  </details>
  
- Inspect the Kibana UI using the link above your terminal. There shouldn't be any logs for now.

- Run `kubectl describe pod -n elastic-stack`

  <details>
  ```
  $ kubectl describe pod -n elastic-stack
  ```
  </details>
  
- Run the command 'kubectl -n elastic-stack exec -it app cat /log/app.log'
  
  <details>
  ```
  $ kubectl -n elastic-stack exec -it app cat /log/app.log
  ```
  </details>
  
- Answer file is located at /var/answers/answer-app.yaml
  
- Inspect the Kibana UI. You should now see logs appearing in the 'Discover' section. You might have to wait for a couple of minutes for the logs to populate. You might have to create an index pattern to list the logs. If not sure check this video: https://bit.ly/2EXYdHf
  
