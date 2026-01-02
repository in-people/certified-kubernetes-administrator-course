# Pre-requisite Switching Routing Gateways (网络基础：交换、路由与网关)

- Take me to [Lecture](https://kodekloud.com/topic/pre-requisite-switching-routing-gateways-cni-in-kubernetes/)

In this section, we will take a look at **Switching, Routing and Gateways** (本文将介绍**交换、路由和网关**的基本概念和操作命令。)

## Switching (交换)

Switching is primarily used for data transmission within a local area network, forwarding packets based on MAC addresses. (交换主要用于局域网内部的数据传输，根据 MAC 地址进行数据包转发。)

### To see the interface on the host system (查看网络接口)

- To see the interface on the host system (查看主机系统上的网络接口：)

```bash
$ ip link
```
- To see the IP Address interfaces. (查看 IP 地址接口：)

```bash
$ ip addr
```

![net-14](../../images/net14.PNG)

## Routing (路由)

Routing is used to determine the path of packets from source to destination, forwarding based on IP addresses. (路由用于确定数据包从源到目的地的路径，基于 IP 地址进行转发。)

### To see the existing routing table on the host system. (查看路由表)

- To see the existing routing table on the host system. (查看主机系统上的现有路由表：)

```bash
$ route
```
```text
controlplane ~ ➜  route  
Kernel IP routing table  
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface  
default         169.254.1.1     0.0.0.0         UG    0      0        0 eth0  
169.254.1.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0  
172.17.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0  
172.17.1.0      172.17.1.0      255.255.255.0   UG    0      0        0 flannel.1  
```
通配符掩码（Wildcard Mask） 是一种特殊的掩码，用于匹配所有可能的IP地址。当子网掩码为 0.0.0.0 时，表示：  
通配符匹配：不进行任何网络地址匹配，**允许所有IP地址通过**  
默认路由：这是默认路由（default route）的标志，表示当没有更具体的路由规则匹配时，所有流量都使用这条路由  
"任意匹配"：二进制表示中，所有位都是0（00000000.00000000.00000000.00000000），意味着不检查任何网络位  

**这是使用 route 命令查看内核IP路由表的输出结果，各列含义如下：**

- **Destination**: 目标网络地址
  - `default`: 默认路由，所有未明确指定路由的流量都通过此路由发送
  - `169.254.1.1`: 特定的主机路由
  - `172.17.0.0` 和 `172.17.1.0`: 特定的网络路由

- **Gateway**: 网关地址
  - `169.254.1.1`: 默认网关，所有外部流量通过此地址转发
  - `0.0.0.0`: 表示目标网络是直连网络，不需要网关

- **Genmask**: 子网掩码
  - `0.0.0.0`: 通配符掩码，用于默认路由
  - `255.255.255.255`: 主机掩码，用于单个主机路由
  - `255.255.255.0`: 标准C类子网掩码(/24)

- **Flags**: 路由标志
  - `U`: Up，路由激活
  - `G`: Gateway，下一跳是网关
  - `H`: Host，目标是单个主机
  - `UH`: 表示该路由是到单个主机的激活路由

- **Iface**: 出站网络接口
  - `eth0`: 以太网接口
  - `cni0`: CNI(Container Network Interface)网络接口
  - `flannel.1`: Flannel虚拟网络接口，常用于Kubernetes集群网络

```text
controlplane ~ ➜  ip route list  
default via 169.254.1.1 dev eth0  
169.254.1.1 dev eth0 scope link  
172.17.0.0/24 dev cni0 proto kernel scope link src 172.17.0.1  
172.17.1.0/24 via 172.17.1.0 dev flannel.1 onlink  
```

**这是使用 ip route list 命令查看路由表的现代方式，输出更简洁：**

- `default via 169.254.1.1 dev eth0`: 默认路由，所有外部流量通过eth0接口发送到网关169.254.1.1
- `169.254.1.1 dev eth0 scope link`: 到特定主机169.254.1.1的路由，通过eth0接口
- `172.17.0.0/24 dev cni0 ...`: 到172.17.0.0/24网络的直连路由，通过cni0接口
- `172.17.1.0/24 via 172.17.1.0 dev flannel.1 ...`: 到172.17.1.0/24网络的路由，通过flannel.1接口和网关172.17.1.0

Or use the modern command: (或者使用现代命令：)

```bash
$ ip route show
```

```bash
$ ip route list
```

### To add entries into the routing table. (添加路由条目)

- To add entries into the routing table. (向路由表添加条目：)

```bash
$ ip route add 192.168.1.0/24 via 192.168.2.1
```

![net-15](../../images/net15.PNG)

## Gateways (网关)

Gateways are devices that connect different networks, typically acting as a default route to forward packets to external networks. (网关是连接不同网络的设备，通常作为默认路由将数据包转发到外部网络。)

### To add a default route. (配置默认路由)

- To add a default route. (添加默认路由：)

```bash
$ ip route add default via 192.168.2.1
```

### To check the IP forwarding is enabled on the host. (启用 IP 转发)

- To check the IP forwarding is enabled on the host. (检查 IP 转发是否在主机上启用：)

```bash
$ cat /proc/sys/net/ipv4/ip_forward
0

$ echo 1 > /proc/sys/net/ipv4/ip_forward
```

**Chinese Explanation (中文说明):**

`/proc/sys/net/ipv4/ip_forward` is a virtual file in the Linux kernel used to control whether the system is allowed to forward IPv4 packets. (`/proc/sys/net/ipv4/ip_forward` 是 Linux 内核中一个虚拟文件，用于控制是否允许系统转发 IPv4 数据包。)

- Value 0: Indicates IP forwarding is disabled (default value). The system only processes packets destined for the local machine and does not forward packets received that are not destined for the local machine. (值为 0：表示禁用 IP 转发（默认值）。此时系统只处理发给本机的数据包，不会将收到的、目的地不是本机的数据包转发出去。)
- Value 1: Indicates IP forwarding is enabled. The system can act as a router, forwarding packets received from one network interface to another network interface (provided the routing table allows it). (值为 1：表示启用 IP 转发。系统可以作为路由器，将从一个网络接口收到的数据包转发到另一个网络接口（前提是路由表允许）。)

### Enable packet forwarding for IPv4. (永久启用 IP 转发)

- Enable packet forwarding for IPv4. (启用 IPv4 数据包转发：)

```bash
$ cat /etc/sysctl.conf

# Uncomment the line
net.ipv4.ip_forward=1
```

- To view the sysctl variables. (查看所有 sysctl 变量：)

```bash
$ sysctl -a 
```

- To reload the sysctl configuration. (重新加载 sysctl 配置：)

```bash
$ sysctl --system
```

## Common Network Commands Explained (常用网络命令详解)

### 🔹 1. ip link
```bash
$ ip link
```
✅ Function: 
Displays status information for all network interfaces (network cards) on the system, including: (显示系统中所有网络接口（网卡）的状态信息，包括：)

- Interface names (e.g., eth0, lo) (接口名称（如 eth0, lo）)
- Whether enabled (UP/Down) (是否启用（UP/Down）)
- MAC address (MAC 地址)
- MTU value (MTU 值)
- Type (e.g., Ethernet) (类型（如 Ethernet）)

📌 Example output snippet: (示例输出片段：)
```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DORMANT group default qlen 1000
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
```

UP indicates the interface is activated; LOWER_UP indicates physical connection is normal. (UP 表示接口已激活；LOWER_UP 表示物理连接正常。)

### 🔹 2. ip addr
```bash
$ ip addr
```
✅ Function:
View IP address configuration for all network interfaces on the current system (IPv4 and IPv6), equivalent to the old ifconfig command. (查看当前系统上所有网络接口的 IP 地址配置（IPv4 和 IPv6），等同于旧命令 ifconfig。)

📌 Output content includes: (输出内容包含：)
- Interface name (e.g., eth0) (接口名（如 eth0）)
- IP address (e.g., 192.168.1.10/24) (IP 地址（如 192.168.1.10/24）)
- Status (whether UP) (状态（是否 UP）)
- MAC address (MAC 地址)

Note: ip addr is the abbreviation for ip a (注意：ip addr 是 ip a 的简写。)

### 🔹 3. ip addr add 192.168.1.10/24 dev eth0
```bash
$ ip addr add 192.168.1.10/24 dev eth0
```
✅ Function:
Add an IPv4 address to the specified network interface eth0. (为指定网络接口 eth0 添加一个 IPv4 地址。)

- 192.168.1.10: IP address assigned to the interface (192.168.1.10：分配给该接口的 IP 地址)
- /24: Subnet mask (i.e., 255.255.255.0) (/24：子网掩码（即 255.255.255.0）)
- dev eth0: Specifies the device name to add the address to (dev eth0：指定要添加地址的设备名称)

📌 Effect:
Allows the host to have this IP on the eth0 interface, enabling communication with other devices. (让主机在 eth0 接口上拥有这个 IP，从而可以与其他设备通信。)

⚠️ If the interface is not enabled (DOWN), you need to execute ip link set eth0 up to activate it. (⚠️ 如果接口未启用（DOWN），需要额外执行 ip link set eth0 up 来激活。)

### 🔹 4. ip route and route
```bash
$ ip route
$ route
```
✅ Function:
View the system's routing table (Routing Table), which determines how packets are forwarded. (查看系统的路由表（Routing Table），决定数据包如何转发。)

- ip route is the modern recommended approach. (ip route 是现代推荐方式。)
- route is the older command with similar functionality but fewer options. (route 是较老的命令，功能类似但选项更少。)

📌 Example output: (示例输出：)
```text
default via 192.168.2.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
192.168.2.0/24 dev eth0 proto kernel scope link src 192.168.2.10
```

This indicates: (这表示：)
- Default gateway is 192.168.2.1 (默认网关是 192.168.2.1)
- Traffic to 192.168.1.0/24 goes through eth0 interface (目标为 192.168.1.0/24 的流量走 eth0 接口)
- Local network segment is reachable via direct connection (本地网段通过直连可达)

### 🔹 5. ip route add 192.168.1.0/24 via 192.168.2.1
```bash
$ ip route add 192.168.1.0/24 via 192.168.2.1
```
✅ Function:
Add a static routing rule to the routing table. (向路由表中添加一条静态路由规则。)

- Access to target network 192.168.1.0/24 (i.e., all hosts with 192.168.1.x) (访问目标网络 192.168.1.0/24（即 192.168.1.x 的所有主机）)
- Needs to go through gateway 192.168.2.1 (需要经过网关 192.168.2.1)
- Usually sent through eth0 (because the gateway is on the local network) (通常是通过 eth0 发送出去（因为该网关在本机所在网络）)

📌 Use case:
When your machine is not in the target network but wants to access it, you need to set up this "jump" path. (当你的机器不在目标网络内，但想访问它时，就需要设置这样的"跳转"路径。)

💡 For example, if you have two switches, each connected to different local area networks, and you want to access one network from the other, you need to set up this route. (💡 比如你有两台交换机，分别连着两个局域网，而你想从一个网访问另一个网，就需要设置这条路由。)

### 🔹 6. cat /proc/sys/net/ipv4/ip_forward
```bash
$ cat /proc/sys/net/ipv4/ip_forward
```
✅ Function:
Read whether IPv4 IP forwarding is enabled on the current system. (读取当前系统是否启用了 IPv4 IP 转发功能。)

- Output of 1: Indicates enabled, allowing the local machine to act as a router to forward packets. (输出为 1：表示已启用，允许本机作为路由器转发数据包。)
- Output of 0: Indicates disabled, cannot forward packets not destined for the local machine. (输出为 0：表示已禁用，不能转发非本机目的的数据包。)

📌 Application significance:
This is a prerequisite for implementing NAT, routing, and gateway functions. For example, when doing NAT or proxy servers, you must ensure this value is 1. (这是实现 NAT、路由、网关功能的前提条件。例如你在做 NAT 或代理服务器时，必须确保此值为 1。)

This setting is only temporarily effective and will be lost after reboot. For permanent effectiveness, modify the /etc/sysctl.conf file and run sysctl -p. (此设置仅临时有效，重启后失效。若需永久生效，应修改 /etc/sysctl.conf 文件并运行 sysctl -p。)

## Summary Table (总结表格)

| Command (命令) | Function (功能) |
|------|------|
| ip link | View network interface status (查看网络接口状态) |
| ip addr | View interface IP address (查看接口 IP 地址) |
| ip addr add ... | Assign IP address to interface (给接口分配 IP 地址) |
| ip route / route | View routing table (查看路由表) |
| ip route add ... | Add static route (添加静态路由) |
| cat /proc/sys/net/ipv4/ip_forward | Check if IP forwarding is enabled (检查是否开启 IP 转发) |

## Practical Application Scenario Example (实际应用场景示例)
Assume you want to turn a Linux host into a simple router: (假设你要把一台 Linux 主机变成一个简单的路由器：)

1. Set IP for eth0: `ip addr add 192.168.1.0/24 dev eth0` (给 eth0 设置 IP：`ip addr add 192.168.1.0/24 dev eth0`)
2. Enable IP forwarding: `echo 1 > /proc/sys/net/ipv4/ip_forward` (启用 IP 转发：`echo 1 > /proc/sys/net/ipv4/ip_forward`)
3. Add default route or specific route: `ip route add 192.168.1.0/24 via 192.168.2.1` (添加默认路由或特定路由：`ip route add 192.168.1.0/24 via 192.168.2.1`)
4. Optional: Configure iptables for NAT (SNAT/DNAT) (可选：配置 iptables 做 NAT（SNAT/DNAT）)

This way, cross-network communication can be achieved! (这样就可以实现跨网络通信了！)
