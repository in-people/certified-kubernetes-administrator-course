# Pre-requisite CoreDNS

  - Take me to [Lecture](https://kodekloud.com/topic/prerequisite-coredns/)

In this section, we will take a look at **CoreDNS**

## Installation of CoreDNS

```
$ wget https://github.com/coredns/coredns/releases/download/v1.7.0/coredns_1.7.0_linux_amd64.tgz
coredns_1.7.0_linux_amd64.tgz

```

## Extract tar file

```
$ tar -xzvf coredns_1.7.0_linux_amd64.tgz
coredns
```

## Run the executable file

- Run the executable file to start a DNS server. By default, it's listen on port 53, which is the default port for a DNS server.

```
$ ./coredns

```

## Configuring the hosts file

- Adding entries into the `/etc/hosts` file.
- CoreDNS will pick the ips and names from the `/etc/hosts` file on the server.

```
$ cat > /etc/hosts
192.168.1.10    web
192.168.1.11    db
192.168.1.15    web-1
192.168.1.16    db-1
192.168.1.21    web-2
192.168.1.22    db-2
```

## Adding into the Corefile

```
$ cat > Corefile
. {
	hosts   /etc/hosts
}

```

## Run the executable file

```
$ ./coredns

```

# CoreDNS 简介

CoreDNS 是一个用 Go 语言编写的、模块化、可插件扩展的 DNS 服务器（Domain Name System server），主要用于将域名解析为 IP 地址。它被设计为轻量、灵活、安全，并且是 Kubernetes 官方推荐并默认使用的集群 DNS 服务（自 Kubernetes v1.13 起取代了 kube-dns）。

## 🔹 CoreDNS 是什么？

- 开源项目：由 CNCF（Cloud Native Computing Foundation）托管，与 Kubernetes 深度集成。
- 核心功能：提供 DNS 解析服务。
- 架构特点：
  - 基于 插件链（plugin chain） 架构：每个功能（如缓存、转发、重写、健康检查等）都是一个插件，按顺序执行。
  - 配置简单：通过一个名为 Corefile 的文本文件定义行为。
  - 支持多种协议：DNS over UDP/TCP，也支持 DNS over TLS/HTTPS（DoT/DoH）。
  - 单二进制文件运行，无外部依赖。

## 🔹 CoreDNS 在 Kubernetes 中的核心作用

在 Kubernetes 集群中，每个 Pod 都需要通过域名访问其他服务（如 my-service.default.svc.cluster.local）。CoreDNS 的主要职责就是：

### ✅ 1. 解析集群内部服务域名

当你在 Pod 中执行：

```bash
nslookup my-app.default.svc.cluster.local
```

CoreDNS 会返回该 Service 对应的 ClusterIP。

域名格式：`<service>.<namespace>.svc.cluster.local`

### ✅ 2. 转发外部域名查询到上游 DNS

对于 www.baidu.com、google.com 等公网域名，CoreDNS 会将请求转发给上游 DNS 服务器（如 /etc/resolv.conf 中配置的 8.8.8.8 或企业 DNS）。

### ✅ 3. 支持 Pod 和 Service 的反向 DNS（PTR 记录）

可用于日志追踪、监控等场景。

### ✅ 4. 提供健康检查和就绪探针

确保 DNS 服务自身高可用。

## 🔹 CoreDNS 的典型配置（Corefile 示例）

Kubernetes 中默认的 Corefile 通常如下：

```corefile
.:53 {
    errors
    health {
        lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

各插件说明：

| 插件 | 作用 |
|------|------|
| kubernetes | 监听 Kubernetes API，自动为 Service 和 Pod 生成 DNS 记录 |
| forward | 将非集群域名（如 baidu.com）转发给上游 DNS（如 /etc/resolv.conf 中的 nameserver） |
| cache | 缓存 DNS 响应，提升性能，减少对外查询 |
| loop | 检测 DNS 循环（防止配置错误导致无限递归） |
| loadbalance | 对多 A 记录做轮询（round-robin），实现客户端负载均衡 |
| prometheus | 暴露指标（如查询次数、延迟），便于监控 |
| health / ready | 提供健康检查端点（/health 和 /ready） |

## 🔹 为什么 Kubernetes 选择 CoreDNS？

| 特性 | 优势 |
|------|------|
| 插件化架构 | 功能按需启用，灵活定制 |
| 单一进程 | 相比旧版 kube-dns（多个容器组合），更简单可靠 |
| 高性能 & 低内存 | Go 语言编写，资源占用少 |
| 活跃社区 & CNCF 毕业项目 | 成熟稳定，广泛采用 |
| 支持现代 DNS 标准 | 如 EDNS、DNSSEC（可选）、IPv6 等 |

## 🔹 如何查看 Kubernetes 中的 CoreDNS？

```bash
# 查看 CoreDNS Pod
kubectl get pods -n kube-system -l k8s-app=kube-dns

# 查看 CoreDNS Service（通常是 10.96.0.10）
kubectl get svc -n kube-system kube-dns

# 查看配置（Corefile）
kubectl -n kube-system get configmap coredns -o yaml
```

所有 Pod 的 /etc/resolv.conf 中的 nameserver 默认指向 kube-dns 的 ClusterIP（如 10.96.0.10）。

## ✅ 总结

| 问题 | 回答 |
|------|------|
| CoreDNS 是什么？ | 一个现代化、插件化的 DNS 服务器 |
| 主要用途？ | 在 Kubernetes 中提供集群内服务发现 + 外部域名解析 |
| 关键能力？ | 自动注册 Service/Pod 域名、转发外部查询、缓存、负载均衡、监控 |
| 配置方式？ | 通过 Corefile 文件定义插件链 |
| 重要性？ | Kubernetes 网络的核心组件之一，没有它，服务间无法通过域名通信 |


#### References Docs

- https://github.com/kubernetes/dns/blob/master/docs/specification.md
- https://coredns.io/plugins/kubernetes/
- https://github.com/coredns/coredns/releases
