# Pre-requisite DNS (DNS 基础知识)

- Take me to [Lecture](https://kodekloud.com/topic/prerequsite-dns/)

In this section, we will take a look at **DNS in the Linux** (本节将介绍 Linux 中的 DNS)

## Name Resolution (名称解析)

- With help of the `ping` command. Checking the reachability of the IP Addr on the Network. (使用 `ping` 命令检查网络上 IP 地址的可达性。)

```bash
$ ping 172.17.0.64
PING 172.17.0.64 (172.17.0.64) 56(84) bytes of data.
64 bytes from 172.17.0.64: icmp_seq=1 ttl=64 time=0.384 ms
64 bytes from 172.17.0.64: icmp_seq=2 ttl=64 time=0.415 ms
```

- Checking with their hostname (使用主机名检查)

```bash
$ ping web
ping: unknown host web
```

- Adding entry in the `/etc/hosts` file to resolve by their hostname. (在 `/etc/hosts` 文件中添加条目以通过主机名解析。)

```bash
$ cat >> /etc/hosts
172.17.0.64  web
# Ctrl + c to exit
```

### 在 /etc/hosts 文件中添加条目 `172.17.0.64  web` 的作用：

建立主机名到IP地址的映射关系，使系统能够通过主机名 `web` 解析到IP地址 `172.17.0.64`。具体作用包括：

- **本地DNS解析**：当执行 `$ ping web` 时，系统会首先查询 `/etc/hosts` 文件，找到 `web` 对应的IP地址 `172.17.0.64`，然后向该IP发送ping请求
- **绕过DNS服务器**：直接在本地完成主机名解析，无需查询外部DNS服务器，提高解析速度
- **解决"unknown host"错误**：之前执行 `$ ping web` 出现 "unknown host web" 错误是因为系统无法解析主机名 `web`，添加映射后就能正确解析
- **临时或永久映射**：为特定主机名建立固定的IP映射关系，常用于开发测试环境或当DNS配置不可用时

### /etc/hosts 文件的作用：

本地主机名解析文件，用于建立主机名（hostname）到IP地址的映射关系。具体功能包括：

- **本地DNS解析**：当系统需要解析主机名时，会首先查询 `/etc/hosts` 文件，如果找到匹配项则直接使用对应的IP地址
- **静态映射**：为特定主机名指定固定的IP地址，优先级高于DNS服务器解析
- **绕过网络DNS查询**：直接在本地完成主机名到IP的转换，提高访问速度，减少网络查询开销
- **开发测试支持**：在开发环境中，将自定义域名映射到本地或测试服务器IP
- **系统故障恢复**：当DNS服务器不可用时，仍可通过hosts文件解析关键主机名
- **网络访问控制**：可以通过将某些域名映射到 `127.0.0.1` 来阻止访问特定网站

文件格式为：`IP地址    主机名    [别名]`

例如：`172.17.0.64  web` 表示将主机名 `web` 解析到IP地址 `172.17.0.64`。

- It will look into the `/etc/hosts` file. (它将查找 `/etc/hosts` 文件。)

```bash
$ ping web
PING web (172.17.0.64) 56(84) bytes of data.
64 bytes from web (172.17.0.64): icmp_seq=1 ttl=64 time=0.491 ms
64 bytes from web (172.17.0.64): icmp_seq=2 ttl=64 time=0.636 ms

$ ssh web

$ curl http://web
```

## DNS

- Every host has a DNS resolution configuration file at `/etc/resolv.conf`. (每台主机都有一个位于 `/etc/resolv.conf` 的DNS解析配置文件。)

```bash
$ cat /etc/resolv.conf
nameserver 127.0.0.53
options edns0
```
```bash
$ cat /etc/resolv.conf 
search qhykafhrxp5zxmwx.svc.cluster.local svc.cluster.local cluster.local us-central1-a.c.kk-lab-prod.internal c.kk-lab-prod.internal google.internal
nameserver 10.96.0.10
options ndots:5
```

### /etc/resolv.conf 文件内容解释：

`/etc/resolv.conf` 文件是Linux系统中DNS解析的配置文件，用于指定域名解析服务器和解析选项。

#### 1. **search** - 搜索域列表
当输入不完整的域名时，系统会按顺序尝试添加这些后缀进行解析。

- 例如：当访问 `web` 时，系统会依次尝试解析：
  - `web.qhykafhrxp5zxmwx.svc.cluster.local`
  - `web.svc.cluster.local`
  - `web.cluster.local`
  - `web.us-central1-a.c.kk-lab-prod.internal`
  - 等等，直到解析成功或遍历完所有搜索域

#### 2. **nameserver** - DNS服务器地址
- `10.96.0.10` 是指定的DNS解析服务器IP地址
- 系统会向此地址发送DNS查询请求
- 在Kubernetes集群中，这通常是集群内部的DNS服务（如CoreDNS）

#### 3. **options** - 解析选项
- `ndots:5` 表示当域名中包含的点号（.）数量少于5个时，优先尝试搜索域解析
  - 例如：`web` 有0个点号，会先尝试 `web` + 搜索域的组合
  - 而 `web.example.com` 有2个点号，仍会先尝试搜索域解析
  - 只有当搜索域解析失败时，才会直接查询原始域名

### 在Kubernetes环境中的特殊性：

- 这个配置是典型的Kubernetes集群内部配置
- `*.cluster.local` 是Kubernetes集群的默认域
- `*.svc.cluster.local` 用于解析服务（Service）名称
- `10.96.0.10` 是集群内部DNS服务的IP地址
- 这个配置确保了在Kubernetes集群中能够正确解析服务名称和外部域名。

- To change the order of dns resolution, we need to do changes into the `/etc/nsswitch.conf` file. (要更改DNS解析顺序，我们需要修改 `/etc/nsswitch.conf` 文件。)

```bash
$ cat /etc/nsswitch.conf

hosts:          files dns
networks:       files
```

- `hosts:` 控制主机名解析（例如 ping example.com 或 gethostbyname() 调用）
- `files dns` 表示：
  - 首先查 本地文件（即 /etc/hosts）
  - 如果没找到，再查 DNS 服务器（通过 /etc/resolv.conf 配置的 DNS）

**/etc/nsswitch.conf 文件说明：**

- 全称为 "Name Service Switch Configuration"（名称服务切换配置）
- 定义了系统如何查找各种类型的信息（如主机名、用户、组等）
- `hosts: files dns` 表示主机名解析的优先级顺序：先查本地文件，再查DNS服务器
- 可以根据需要调整顺序，如 `hosts: dns files` 会优先查询DNS服务器

- If it fails in some conditions. (如果在某些情况下失败。)

```bash
$ ping wwww.github.com
ping: www.github.com: Temporary failure in name resolution
```

- Adding well known public nameserver in the `/etc/resolv.conf` file. (在 `/etc/resolv.conf` 文件中添加知名公共域名服务器。)

```bash
$ cat /etc/resolv.conf
nameserver   127.0.0.53
nameserver   8.8.8.8
options edns0
```

```bash
$ ping www.github.com
PING github.com (140.82.121.3) 56(84) bytes of data.
64 bytes from 140.82.121.3 (140.82.121.3): icmp_seq=1 ttl=57 time=7.07 ms
64 bytes from 140.82.121.3 (140.82.121.3): icmp_seq=2 ttl=57 time=5.42 ms
```


## Domain Names (域名)

![net-8](../../images/net8.PNG)

## Search Domain (搜索域)

![net-9](../../images/net9.PNG)

## Record Types (记录类型)

![net-10](../../images/net10.PNG)

## Networking Tools (网络工具)

- Useful networking tools to test dns name resolution. (用于测试DNS名称解析的有用网络工具。)

#### nslookup 

```bash
$ nslookup www.baidu.com
;; Got recursion not available from 10.96.0.10
;; Got recursion not available from 10.96.0.10
;; Got recursion not available from 10.96.0.10
Server:         10.96.0.10
Address:        10.96.0.10#53

Non-authoritative answer:
www.baidu.com   canonical name = www.a.shifen.com.
www.a.shifen.com        canonical name = www.wshifen.com.
Name:   www.wshifen.com
Address: 103.235.46.102
Name:   www.wshifen.com
Address: 103.235.46.115
```

### nslookup 输出解释：

#### 🔹 1. ;; Got recursion not available from 10.96.0.10（重复三次）

**✅ 含义：**

这是警告信息，表示你的 DNS 客户端（nslookup）向 DNS 服务器 10.96.0.10 请求"递归查询"（recursive query），但该服务器拒绝提供递归服务。

**📌 什么是递归查询？**

- 正常情况下，当你问 DNS 服务器 "www.baidu.com 的 IP 是什么？"，它应该帮你一路查到底（问根服务器 → .com 服务器 → baidu.com 服务器），最后把答案返回给你。这就是"递归"。
- 但某些 DNS 服务器（尤其是权威 DNS 服务器或配置为非递归的服务器）只回答自己"直接知道"的内容，不会帮你去查别的域名——这叫"非递归"（non-recursive）。

**❓为什么还能得到结果？**

虽然服务器说"不支持递归"，但它恰好缓存了 www.baidu.com 的记录（或者它是上游转发器），所以最终还是返回了答案。否则你会看到 *** Can't find ...: No answer.

**💡 在 Kubernetes 环境中，**10.96.0.10 通常是 CoreDNS 服务的 ClusterIP。默认情况下 CoreDNS 是支持递归的，但如果配置错误、资源不足或网络策略限制，也可能出现此警告。

#### 🔹 2. Server: 10.96.0.10 和 Address: 10.96.0.10#53

- 表示你当前使用的 DNS 服务器是 10.96.0.10，监听在标准 DNS 端口 53。
- 这个 IP 很典型：在 Kubernetes 集群中，kube-dns 或 CoreDNS 服务通常被分配为 10.96.0.10（可通过 kubectl get svc -n kube-system 查看）

#### 🔹 3. 非权威应答（Non-authoritative answer）

```
Non-authoritative answer:
www.baidu.com   canonical name = www.a.shifen.com.
www.a.shifen.com        canonical name = www.wshifen.com.
Name:   www.wshifen.com
Address: 103.235.46.102
Name:   www.wshifen.com
Address: 103.235.46.115
```

**✅ 解释：**

- **Non-authoritative answer：** 表示这个 DNS 答案不是来自百度官方的权威 DNS 服务器，而是来自一个缓存服务器（比如你的本地 DNS、ISP DNS 或 Kubernetes 的 CoreDNS）。这是正常现象。
- **CNAME 链：**
  - www.baidu.com 是一个别名（CNAME），指向 www.a.shifen.com
  - www.a.shifen.com 又是一个别名，指向 www.wshifen.com
  - 最终 www.wshifen.com 有两条 A 记录（IPv4 地址）：
    - 103.235.46.102
    - 103.235.46.115
- **✅ 所以你最终访问的是 103.235.46.102 或 103.235.46.115，这是百度的真实服务器 IP。**

#### 🔍 总结：整个过程发生了什么？

1. 你的机器向 DNS 服务器 10.96.0.10（很可能是 Kubernetes 的 CoreDNS）发起查询。
2. CoreDNS 尝试递归查询 www.baidu.com，但可能由于配置或网络问题，没有正确启用递归，于是返回了警告。
3. 但幸运的是，CoreDNS 已经缓存了该记录（或通过上游转发成功获取），所以仍然返回了正确的 IP。
4. 你得到了百度的两个真实 IP 地址，可以正常访问。

#### dig

```bash
dig www.baidu.com

; <<>> DiG 9.18.30-0ubuntu0.22.04.2-Ubuntu <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30407
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e37cc10b2067fe0a (echoed)
;; QUESTION SECTION:
;www.baidu.com.                 IN      A

;; ANSWER SECTION:
www.baidu.com.          30      IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       30      IN      CNAME   www.wshifen.com.
www.wshifen.com.        30      IN      A       103.235.46.102
www.wshifen.com.        30      IN      A       103.235.46.115

;; Query time: 244 msec
;; SERVER: 10.96.0.10#53(10.96.0.10) (UDP)
;; WHEN: Fri Jan 02 03:04:35 UTC 2026
;; MSG SIZE  rcvd: 204

```

成功解析了百度的域名。下面是对这个 dig 输出结果的逐部分详细解释。

### dig 输出解释：

#### 🔹 1. 整体状态：status: NOERROR

- 表示 DNS 查询成功完成，没有错误。
- 与之前 baidu.google.com 返回的 NXDOMAIN（域名不存在）形成鲜明对比。

#### 🔹 2. FLAGS 解读：qr rd ra

- qr：这是一个应答（Query Response）
- rd：客户端请求时要求递归查询（Recursion Desired）
- ra：DNS 服务器支持并执行了递归（Recursion Available）
- ✅ 说明你的 DNS 服务器（10.96.0.10，即 Kubernetes 的 CoreDNS）正常工作，并帮你完成了完整的递归解析。

#### 🔹 3. QUESTION SECTION（查询的问题）

```
;www.baidu.com.                 IN      A
```

- 你问的是：www.baidu.com 的 IPv4 地址（A 记录）是什么？

#### 🔹 4. ANSWER SECTION（核心答案）

```
www.baidu.com.          30      IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       30      IN      CNAME   www.wshifen.com.
www.wshifen.com.        30      IN      A       103.235.46.102
www.wshifen.com.        30      IN      A       103.235.46.115
```

**✅ 分步解析过程：**

- **www.baidu.com 是一个 CNAME（别名）**
  - 它不直接对应 IP，而是指向另一个域名：www.a.shifen.com
  - TTL = 30 秒（缓存时间较短，便于负载均衡或故障切换）

- **www.a.shifen.com 也是一个 CNAME**
  - 再次跳转到 www.wshifen.com

- **www.wshifen.com 有两条 A 记录**
  - 最终解析出两个 IPv4 地址：
    - 103.235.46.102
    - 103.235.46.115

**💡 这是典型的 CDN 或负载均衡架构：通过多层 CNAME 将流量导向不同的服务器集群。**

#### 🔹 5. AUTHORITY SECTION：为空（AUTHORITY: 0）

- 因为返回的是非权威应答（来自缓存或递归解析器，如 CoreDNS），所以没有权威服务器信息。
- 如果你直接向百度的权威 DNS（如 ns1.baidu.com）查询，这里会显示 NS 记录。

#### 🔹 6. 其他关键信息

| 字段 | 值 | 说明 |
|------|------|------|
| SERVER | 10.96.0.10#53 | 使用的 DNS 服务器是 Kubernetes 内部的 CoreDNS |
| Query time | 244 msec | 查询耗时 244 毫秒（略高，可能因跨公网或 CoreDNS 转发延迟） |
| WHEN | Fri Jan 02 03:04:35 UTC 2026 | 查询发生的时间 |
| MSG SIZE rcvd | 204 | 收到的 DNS 报文大小（字节） |

#### ✅ 总结：整个解析流程

1. 你的 Pod 向 CoreDNS（10.96.0.10）发起查询：www.baidu.com 的 A 记录？
2. CoreDNS 递归查询根服务器 → .com 服务器 → baidu.com 权威服务器
3. 权威服务器返回 CNAME 链：
   - www.baidu.com → www.a.shifen.com → www.wshifen.com
4. 继续解析 www.wshifen.com，得到两个 A 记录
5. CoreDNS 将完整结果返回给你
6. ✅ 最终你获得了百度的真实 IP 地址，可以正常访问。

## Summary (总结)

This document covers the fundamental concepts of DNS resolution in Linux:

1. **Name Resolution Process**: How hostnames are resolved to IP addresses
2. **Local Resolution**: Using `/etc/hosts` for static hostname-to-IP mapping
3. **DNS Configuration**: The role of `/etc/resolv.conf` and `/etc/nsswitch.conf`
4. **DNS Tools**: Using `nslookup` and `dig` for DNS troubleshooting

本文档涵盖了Linux中DNS解析的基本概念：

1. **名称解析过程**：主机名如何解析为IP地址
2. **本地解析**：使用 `/etc/hosts` 进行静态主机名到IP的映射
3. **DNS配置**：`/etc/resolv.conf` 和 `/etc/nsswitch.conf` 的作用
4. **DNS工具**：使用 `nslookup` 和 `dig` 进行DNS故障排除
