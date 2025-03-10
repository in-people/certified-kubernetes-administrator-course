# Practice Test - Resource Limits
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-resource-limits/)
  
Solutions to practice test - resource limtis
- Run the command 'kubectl describe pod rabbit' and inspect requests.
  
  <details>

  ```
  $ kubectl describe pod rabbit
  ```
  </details>

- Run the command 'kubectl delete pod rabbit'.

  <details>

  ```
  $ kubectl delete pod rabbit
  ```
  </details>

- Run the command 'kubectl get pods' and inspect the status of the pod elephant

  <details>

  ```
  $ kubectl get pods
  ```
  </details>

- The status 'OOMKilled' indicates that the pod ran out of memory. 内存用尽导致pod起不来。
-  Identify the memory limit set on the POD.
```yaml
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Mon, 10 Mar 2025 11:35:19 +0000
      Finished:     Mon, 10 Mar 2025 11:35:20 +0000
```

- Generate a template of the existing pod.

  <details>

  ```
  $ kubectl get pods elephant -o yaml > elephant.yaml
  ```
  </details>

  Update the elephant.yaml pod definition with the resource memory limits to 20Mi
  
  <details>

  ```
  resources:
      limits:
        memory: 20Mi
  ---
  ```
  
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: elephant
  name: elephant
spec:
  containers:
  - image: polinux/stress
    name: elephant
    resources:
       limits:
         memory: 20Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

  </details>

  Delete the pod and recreate it.
  
  <details>

  ```
  $ kubectl delete pod elephant
  $ kubectl create -f elephant.yaml
  ```
  </details>

- Inspect the status of POD. Make sure it's running
  
  <details>

  ```
  $ kubectl get pods
  ```
  </details>

- Run the command 'kubectl delete pod elephant'.

  <details>

  ```
  $ kubectl delete pod elephant
  ```
  </details>



