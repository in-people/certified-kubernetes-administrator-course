# Practice Test - Network Policies
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-network-policies/)

Solutions to practice test - network policies

- Run the command 'kubectl get networkpolicy'
  
  <details>
  
  ```
  $ kubectl get networkpolicy
  ```
  
  </details>
  
- Run the command 'kubectl get networkpolicy'
   
  <details>
  
  ```
  $ kubectl get networkpolicy
  ```
  
  </details>
  
- Run the command 'kubectl get networkpolicy' and look under pod selector
  
  <details>
  
  ```
  $ kubectl get networkpolicy
  ```
  
  </details>
  
- Run the command 'kubectl describe networkpolicy' and look under PolicyTypes
  
  <details>
  
  ```
  $ kubectl describe networkpolicy
  ```
  
  </details>
  
- What is the impact of the rule configured on this Network Policy?
  
  <details>
  
  ```
  Traffic from internal to payroll pod is blocked
  ```
  
  </details>
  
- What is the impact of the rule configured on this Network Policy?
  
  <details>
  
  ```
  Internal pod can access port 8080 on payroll pod
  ```
  
  </details>
  
- Access the UI of these applications using the link given above the terminal.

- Only internal applications can access payroll service

- Perform a connectivity test using the User Interface of the Internal Application to access the 'external-service' at port '8080'.
  
  <details>
  
  ```
  Successful
  ```
  
  </details>
  
- Answer file located at /var/answers/answer-internal-policy.yaml
  
  <details>
  
  ```
  $ kubectl create -f /var/answers/answer-internal-policy.yaml
  ```
  
  </details>

```json
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy # 修正1：缩进对齐，删除多余空格
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal # 修正2：移除了这里的 “-” 错误标记
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - port: 8080
      protocol: TCP
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - port: 3306
      protocol: TCP
  # 修正3 & 4：新增独立的DNS规则块，并修正协议重复错误
  - ports: # 这是一个独立的egress规则项，需要添加 “-” 并用 “to:” 或保留为空
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP # 将第二个UDP改为TCP，DNS需要两者
 ```
  
