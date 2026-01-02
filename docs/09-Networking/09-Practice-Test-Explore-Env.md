# Practice Test - Explore Env

  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-explore-environment/)

#### Solution

1. <details>
   <summary>How many nodes are part of this cluster?</summary>

      ```
      kubectl get nodes
      ```

      Count the results

   </details>

1. <details>
   <summary>What is the Internal IP address of the controlplane node in this cluster?</summary>

      ```
      kubectl get nodes -o wide
      ```

      Note the value in `INTERNAL-IP` column for `controlplane`

   </details>

1. <details>
   <summary>What is the network interface configured for cluster connectivity on the controlplane node?</summary>

   This will be the network interface that has the same IP address you determined in the previous question.

   ```
   ip a
   ```

   There is quite a lot of output for the above command. We can filter it better:

   ```
   ip a | grep -B2 X.X.X.X
   ```

   where `X.X.X.X` is the IP address you got from the previous question. `grep -B2` will find the line containing the value we are looking for and print that and the previous 2 line of output. It will look like this, though the values will be different each time you run the lab.

   ```
   3058: eth0@if3059: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
      link/ether 02:42:c0:08:ea:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
     inet 192.8.234.3/24 brd 192.8.234.255 scope global eth0
   ```

   From this, we can determine the answer to be

   > `eth0`

   </details>

1. <details>
   <summary>What is the MAC address of the interface on the controlplane node?</summary>

   This value is also present in the output of the command you ran for the previous question. The MAC address is the value in the `link/ether` field of the output and is 6 hex numbers separated by `:`. Note that the value can be different each time you run the lab.

   If the output for `eth0` is

   ```
   3058: eth0@if3059: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default
      link/ether 02:42:c0:08:ea:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
     inet 192.8.234.3/24 brd 192.8.234.255 scope global eth0
   ```

   then the MAC address is

   > `02:42:c0:08:ea:03`

   </details>

1. <details>
   <summary>What is the IP address assigned to node01?</summary>

   ```
   kubectl get nodes -o wide
   ```

   Note the value in `INTERNAL-IP` column for `node01`

   </details>

1. <details>
   <summary>What is the MAC address assigned to node01?</summary>

   For this we will need to SSH onto `node01` so we can view its interfaces. We know what IP to look for, as we determined this in the previous question

   ```
   ssh node01
   ip a | grep -B2 X.X.X.X
   ```

   where `X.X.X.X` is the IP address you got from the previous question. Again, look at the `link/ether` field.

   We could guess that the correct interface on `node01` is also `eth0` and simply run

   ```
   ip link show eth0
   ```

   but it's best to be sure.

   Now return to `controlplane`

   ```
   exit
   ```

   </details>

1. <details>
   <summary>We use Containerd as our container runtime. What is the interface/bridge created by Containerd on this host?</summary>

   This is not immediately straight forward.

   ```
   ip link show
   ```

   Know that

   * Any interface with name beginning `eth` is a "physical" interface, and represents a network card attached to the host.
   * Interface `lo` is the loopback, and covers all IP addresses starting with `127`. Every computer has this.
   * Any interface with name beginning `veth` is a virtual network interface used for tunnelling between the host and the pod network. These connect with bridges, and the bridge interface name is listed with their details.

   We can see that for the two `veth` devices, they are associated with another device in the list `cni0`, therefore that is the answer.

   </details>

1. <details>
   <summary>What is the state of the interface cni0?</summary>

   You can see in the output of the previous command that the state field for `cni0` is

   > `UP`

   </details>

1. <details>
   <summary>If you were to ping google from the controlplane node, which route does it take?</summary>

   What is the IP address of the Default Gateway?

   Run

   ```
   ip route show default
   ```

   and note the output

   </details>

1. <details>
   <summary>What is the port the kube-scheduler is listening on in the controlplane node?</summary>

   Use the [netstat](https://linux.die.net/man/8/netstat) command to look at network sockets used by programs running on the host. There's a lot of output, so we will filter by process name, i.e. `kube-scheduler`

   ```
   netstat -nplt | grep kube-scheduler
   ```

   <details>
   <summary>What the netstat options used mean</summary>

   * `-n` - Show IP addresses (don't try to resolve to host names)
   * `-p` - Show the process names (e.g. `kube-scheduler`)
   * `-l` - Include only _listening_ sockets
   * `-t` - Include only TCP sockets

   </details>

   Output:

   ```
   tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3291/kube-scheduler
   ```

   We can see it's listening on localhost, port `10259`
   </details>

1. <details>
   <summary>Notice that ETCD is listening on two ports. Which of these have more client connections established?</summary>

   We use `netstat` with slightly different options and filter for `etcd`

   ```
   netstat -anp | grep etcd
   ```
```bash
netstat -anp | grep etcd
```

```bash
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      2811/etcd           
tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      2811/etcd           
tcp        0      0 192.168.126.139:2380    0.0.0.0:*               LISTEN      2811/etcd           
tcp        0      0 192.168.126.139:2379    0.0.0.0:*               LISTEN      2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:37874         ESTABLISHED 2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:37888         ESTABLISHED 2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:37636         ESTABLISHED 2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:37700         ESTABLISHED 2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:37556         ESTABLISHED 2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:38558         ESTABLISHED 2811/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:37918         ESTABLISHED 2811/etcd
```

请问这是什么意思？

## 你运行的命令：

```bash
netstat -anp | grep etcd
```

输出显示了 etcd 进程（PID 2811）当前监听的端口和建立的连接。下面逐行解释其含义，并分析你的 etcd 配置状态。

## 🔍 命令说明

netstat -anp：
- -a：显示所有连接和监听端口
- -n：以数字形式显示地址/端口（不解析域名或服务名）
- -p：显示进程 PID 和名称（需 root 权限才能看到全部）
- grep etcd：只过滤包含 "etcd" 的行

## ✅ 输出解析

### 🟢 1. 监听（LISTEN）状态的端口

| 行 | 含义 |
|----|------|
| tcp 0 0 127.0.0.1:2379 0.0.0.0:* LISTEN 2811/etcd | 仅本地回环监听 2379 → 只允许本机程序（如 kube-apiserver）通过 127.0.0.1 访问 etcd 客户端接口 |
| tcp 0 0 127.0.0.1:2381 0.0.0.0:* LISTEN 2811/etcd | 监听本地 2381 端口（非标准端口，可能是自定义配置或备用端口） |
| tcp 0 0 192.168.126.139:2380 0.0.0.0:* LISTEN 2811/etcd | Peer 端口对外监听 → 其他 etcd 节点可通过 192.168.126.139:2380 与它通信（用于集群同步） |
| tcp 0 0 192.168.126.139:2379 0.0.0.0:* LISTEN 2811/etcd | 客户端端口对外监听 → 允许其他机器（如另一个 Master 节点）直接访问此 etcd 实例 |

📌 关键 IP：192.168.126.139 是这台主机的内网 IP。

### 🔵 2. 已建立（ESTABLISHED）的连接

多行类似：

```text
tcp 0 0 127.0.0.1:2379 127.0.0.1:37874 ESTABLISHED 2811/etcd
```

- 表示 本机有其他进程正在连接 127.0.0.1:2379（即 etcd 的客户端接口）
- 源端口如 37874、37888 等是临时端口（由客户端随机选择）
- 典型客户端：
  - kube-apiserver
  - etcdctl（手动执行命令时）
  - 监控工具（如 Prometheus）

✅ 这些连接正常，说明 Kubernetes 控制平面组件正在使用 etcd。

   <details>
   <summary>What the netstat options used mean</summary>

   * `-a` - Include sockets in all states
   * `-n` - Show IP addresses (don't try to resolve to host names)
   * `-p` - Show the process names (e.g. `etcd`)

   </details>

   You can see that by far and away, the most used port is `2379`.

   </details>

1. Information

   That's because `2379` is the port of ETCD to which API server connects to There are multiple concurrent connections so that API server can process multiple etcd operations simultaneously.

    `2380` is only for etcd peer-to-peer connectivity when you have multiple controlplane nodes. In this case we don't.



