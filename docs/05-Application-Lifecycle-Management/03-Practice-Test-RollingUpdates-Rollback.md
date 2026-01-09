# Practice Test - Rolling Updates and Rollback
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-rolling-updates-and-rollbacks/)
  
Solutions to practice test - rolling updates and rollback
- We have deployed a simple web application. Inspect the PODs and the Services

  <details>
  
  ```
  $ kubectl get pods
  $ kubectl get services
  ```
  </details>
  
- What is the current color of the web application?
  
  <details>
  
  ```
  Access the web portal
  ```
  </details>
    
- Execute the script at /root/curl-test.sh.

- Run the command 'kubectl describe deployment' and look at 'Desired Replicas'

  <details>
  
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- Run the command 'kubectl describe deployment' and look for 'Images'
  
  <details>
  
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- Run the command 'kubectl describe deployment' and look at 'StrategyType'
  
  <details>
  
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- If you were to upgrade the application now what would happen?
  
  <details>
  
  ```
  PODs are upgraded few at a time
  ```
  </details>
  
- Run the command 'kubectl edit deployment frontend' and modify the required feild
  
  <details>
  
  ```
  $ kubectl edit deployment frontend
  ```
  </details>
    
- Execute the script at /root/curl-test.sh.

- Look at the Max Unavailable value under RollingUpdateStrategy in deployment details

  <details>
  ```
  $ kubectl describe deployment
  ```
  </details>
  
- Run the command 'kubectl edit deployment frontend' and modify the required field. Make sure to delete the properties of rollingUpdate as well, set at 'strategy.rollingUpdate'.
  
  <details>
  
  ```
  $ kubectl edit deployment frontend
  ```
  
  </details>
  
- Run the command 'kubectl edit deployment frontend' and modify the required feild

  <details>
  
  ```
  $ kubectl edit deployment frontend
  ```
  </details>
  
- Execute the script at /root/curl-test.sh.

| 策略类型                  | 行为                                             | 是否支持零停机               | 适用场景                                              |
| ------------------------- | ------------------------------------------------ | ---------------------------- | ----------------------------------------------------- |
| `RollingUpdate`（默认） | 逐步用新 Pod 替换旧 Pod，过程中保持部分 Pod 运行 | ✅ 是（配合 readinessProbe） | 生产环境、需要高可用的服务                            |
| `Recreate`              | 先全部删除旧 Pod，再创建新 Pod                   | ❌ 否（服务会短暂中断）      | 有状态应用、开发/测试环境、无法并行运行新旧版本的场景 |



