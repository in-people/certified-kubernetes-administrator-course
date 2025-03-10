# Practice Test - DaemonSets
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-daemonsets/)
  
Solutions to practice test daemonsets
- Run the command kubectl get daemonsets --all-namespaces
  
  <details>

  ```
  $ kubectl get daemonsets --all-namespaces
  ```
  </details>

- Run the command kubectl get daemonsets --all-namespaces

  <details>

  ```
  $ kubectl get daemonsets --all-namespaces
  ```
  </details>

- Run the command kubectl get all --all-namespaces and identify the types

  <details>

  ```
  $ kubectl get all --all-namespaces
  ```
  </details>

- Run the command kubectl describe daemonset kube-proxy --namespace=kube-system

  <details>

  ```
  $ kubectl describe daemonset kube-proxy --namespace=kube-system
  ```
  </details>

- Run the command kubectl describe daemonset kube-flannel-ds-amd64 --namespace=kube-system

  <details>

  ```
  $ kubectl describe daemonset kube-flannel-ds-amd64 --namespace=kube-system
  ```
  </details>
    
- Create a daemonset
使用kubectl create ... -o yaml快速创建一个yaml，再修改成daemonset yaml
```shell
  An easy way to create a DaemonSet is to first generate a YAML file for a Deployment with the command
    kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20
   -n kube-system --dry-run=client -o yaml > fluentd.yaml. Next,
    remove the replicas, strategy and status fields from the YAML file using a text editor.
   Also, change the kind from Deployment to DaemonSet.
Finally, create the Daemonset by running kubectl create -f fluentd.yaml

```
  <details>

  ```
  $ vi ds.yaml
  ```

  ```
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: elasticsearch
    namespace: kube-system
    labels:
      k8s-app: fluentd-logging
  spec:
    selector:
      matchLabels:
        name: elasticsearch
    template:
      metadata:
        labels:
          name: elasticsearch
      spec:
        tolerations:
        # this toleration is to have the daemonset runnable on master nodes
        # remove it if your masters can't run pods
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
        containers:
        - name: elasticsearch
          image: k8s.gcr.io/fluentd-elasticsearch:1.20
  ```
  </details>

- To create the daemonset and list the daemonsets and pods

  <details>

  ```
  $ kubectl create -f ds.yaml
  $ kubectl get ds -n kube-system
  $ kubectl get pod -n kube-system|grep elasticsearch
  ```
  </details>




