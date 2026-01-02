# Practice Test Networking Weave

  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-networking-weave/)

#### Solution 

  1. <details>
      <summary>How many Nodes are part of this cluster?</summary>

      ```bash
      kunbectl get nodes
      ```

      > 2

     </details>

  2. <details>
      <summary>What is the Networking Solution used by this cluster?</summary>

      Two ways to do this:

      1.
         ```bash
         kubectl get pods -n kube-system
         ```

      1.
         ```bash
         ls -l /opt/cni/bin
         ```

      In both you see evidence of

      > weave

     </details>

  3. <details>
      <summary>How many weave agents/peers are deployed in this cluster?</summary>

      ```bash
      kubectl get pods -n kube-system
      ```

      > 2

     </details>

  4. <details>
      <summary>On which nodes are the weave peers present?</summary>

      ```bash
      kubectl get pods -n kube-system -o wide
      ```

      > One on every node

     </details>

  5. <details>
      <summary>Identify the name of the bridge network/interface created by weave on each node.</summary>

      At either host...

      ```bash
      ip addr list
      ```

      > weave

      In actual fact, the network interface is `weave` and the bridge is implemented by `vethwe-datapath@vethwe-bridge` and `vethwe-bridge@vethwe-datapath`

     </details>

  6. <details>
      <summary>What is the POD IP address range configured by weave?</summary>

      Examine output of previous connad for `weave` interface. Note its IP begins with `10.`, so...

      > 10.X.X.X

     </details>

  7. <details>
      <summary>What is the default gateway configured on the PODs scheduled on node01?</summary>

      Now we can deduce this from the naswer to the previous question. Since we know weave's IP range, its gateway must be on the same network. However we can verify that by starting a pod which is known to contain the `ip` tool.

      Remember this [container image](https://github.com/wbitt/Network-MultiTool). It is extremely useful for debugging cluster networking issues!

      ```bash
      kubectl run testpod --image=wbitt/network-multitool
      ```

      Wait for it to be running.

      ```bash
      kubectl exec -it testpod -- ip route
      ```

      Note the first line of the output. This is the answer.
     </details>

# 网络策略测试与 CNI 更换指南

## 问题1：网络策略测试

在集群的 default 命名空间中已部署了两个应用：frontend（前端）和 backend（后端）。同时，已创建了一个名为 deny-backend 的网络策略（NetworkPolicy），用于阻止访问 backend 应用。

为了测试该策略是否生效，请使用以下命令：

```bash
kubectl exec -it frontend -- curl -m 5 <BACKEND-POD-IP>
```

你需要将 `<BACKEND-POD-IP>` 替换为 backend Pod 的实际 IP 地址。可通过以下命令获取该 IP：

```bash
kubectl get pods -o wide
```

请问：frontend Pod 能否成功访问 backend Pod？

```bash
kubectl get pod -o wide
```

```
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
backend    1/1     Running   0          3m48s   172.17.0.5   controlplane   <none>           <none>
frontend   1/1     Running   0          3m48s   172.17.0.4   controlplane   <none>           <none>
```

```bash
controlplane /etc/cni/net.d ➜  kubectl exec -it frontend -- curl -m 5 172.17.0.5
```

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>

```

## 问题2：删除 Flannel CNI

The Flannel CNI does not support NetworkPolicies.

Delete Flannel CNI.

```bash
kubectl delete daemonset -n kube-flannel kube-flannel-ds
kubectl delete cm kube-flannel-cfg -n kube-flannel
rm /etc/cni/net.d/10-flannel.conflist
```

## 问题3：安装 Calico CNI

### 安装 Calico CNI

First, install the Calico CRDs:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/operator-crds.yaml
```

Then install the Calico operator:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/tigera-operator.yaml
```

Download the custom resource definition yaml:

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/custom-resources.yaml -O
```

Edit the file to update the CIDR field to 172.17.0.0/16 for pod communication to work successfully on the cluster:

```bash
# Edit custom-resources.yaml and update the cidr field
```

Edit the file to update the CIDR network. The file should look like this:

```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 172.17.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
---
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```

Apply the manifest file:

```bash
kubectl apply -f custom-resources.yaml
```

Wait until the calico pods are running:

```bash
watch kubectl get pods -A
```

You may ignore errors on the csi-node-driver pod deployed in the calico-system namespace.
