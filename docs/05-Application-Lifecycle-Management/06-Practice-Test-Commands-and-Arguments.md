# Practice Test - Commands and Arguments
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-commands-and-arguments/)
  
Solutions to practice test - commands and arguments
- Run the command 'kubectl get pods' and count the number of pods.
  
  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
  
- Run the command 'kubectl describe pod' and look for command option

  <details>
  
  ```
  $ kubectl describe pod
  ```
  </details>
  
- Set the command option to ['sleep', '5000']. Answer file at: /var/answers/answer-ubuntu-sleeper-2.yaml

  - command:  
    - sleep  
    - "4800"  

apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "5000"]

- Both sleep and 1200 should be defined as a string. Answer file at: /var/answers/answer-ubuntu-sleeper-3.yaml

apiVersion: v1  
kind: Pod  
metadata:  
  name: ubuntu-sleeper-3  
spec:  
  containers: 
  - name: ubuntu  
    image: ubuntu   
    command:  
      - "sleep"    
      - "1200"  

- Answer file at: /var/answers/answer-ubuntu-sleeper-3-2.yaml

- Inspect the file 'Dockerfile' given at /root/webapp-color. What command is run at container startup?
  
  <details>
  
  ```
  python app.py
  ```
  </details>
  
- Inspect the file 'Dockerfile2' given at /root/webapp-color. What command is run at container startup?
**python app.py --color red**

  <details>
  ```
  python app.py --color red
  ```
  </details>
  
- The 'command' (entrypoint) is overridden in the pod definition. So the answer is --color green

- Inspect the two files under directory 'webapp-color-3'. What command is run at container startup?

  <details>
  
  ```
  python app.py --color pink
  ```

command: ["python", "app.py"]  
This configuration explicitly overrides the ENTRYPOINT specified in the Dockerfile. Therefore, the container will initiate with python app.py as its base command.  

args: ["--color", "pink"]  
These arguments are appended to the command specified in the Pod definition.  
  </details>
  
- Answer file located at /var/answers/answer-webapp-color-green.yaml


