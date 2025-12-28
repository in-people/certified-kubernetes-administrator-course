
Can the kubectl scale command be used to scale down a statefulset in Kubernetes?
Yes

What component in a Kubernetes cluster is responsible for providing metrics to the HPA?  
metrics server

controlplane ~ ➜  kubectl get hpa nginx-deployment  
NAME               REFERENCE                     TARGETS              MINPODS   MAXPODS   REPLICAS   AGE  
nginx-deployment   Deployment/nginx-deployment   cpu: <unknown>/80%   1         3         3          101s  
The HPA status shows /80 for the CPU target. what could be a possible reason?  
"The deployment does not have any resource fields defined."  

✖ kubectl events hpa nginx-deployment | grep -i "ScalingReplicaSet"  
9m13s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-bf744486c from 0 to 7  
5m55s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 7 to 3  
105s                    Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-bf744486c from 3 to 7  
105s                    Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-6c78464d7c from 0 to 2  
104s                    Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-6c78464d7c from 2 to 3  
104s                    Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 7 to 6  
100s                    Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 6 to 3  
99s                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 3 to 2  
92s (x4 over 99s)       Normal    ScalingReplicaSet              Deployment/nginx-deployment                (combined from similar events): Scaled down replica set nginx-deployment-bf744486c from 1 to 0  
55s (x2 over 100s)      Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-6c78464d7c from 3 to 1  




 kubectl events hpa nginx-deploymenet
LAST SEEN               TYPE      REASON                         OBJECT                                     MESSAGE
12m (x2 over 12m)       Normal    Sync                           Ingress/flask-ingress                      Scheduled for sync
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-m4dbn
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-m4dbn       Successfully assigned default/nginx-deployment-bf744486c-m4dbn to node01
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-6r8ck
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-dh2hp
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-5462v       Successfully assigned default/nginx-deployment-bf744486c-5462v to node01
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-dldmx
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-sz5l2
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-bkl8w
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-6r8ck       Successfully assigned default/nginx-deployment-bf744486c-6r8ck to node01
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-bkl8w       Successfully assigned default/nginx-deployment-bf744486c-bkl8w to node01
10m                     Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-5462v
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-dh2hp       Successfully assigned default/nginx-deployment-bf744486c-dh2hp to node01
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-sz5l2       Successfully assigned default/nginx-deployment-bf744486c-sz5l2 to node01
10m                     Normal    Scheduled                      Pod/nginx-deployment-bf744486c-dldmx       Successfully assigned default/nginx-deployment-bf744486c-dldmx to node01
10m                     Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-bf744486c from 0 to 7
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-bkl8w       Pulling image "nginx:1.14.2"
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-dldmx       Pulling image "nginx:1.14.2"
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-5462v       Pulling image "nginx:1.14.2"
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-dh2hp       Pulling image "nginx:1.14.2"
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-m4dbn       Pulling image "nginx:1.14.2"
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-6r8ck       Pulling image "nginx:1.14.2"
10m                     Normal    Pulling                        Pod/nginx-deployment-bf744486c-sz5l2       Pulling image "nginx:1.14.2"
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-dh2hp       Successfully pulled image "nginx:1.14.2" in 199ms (5.805s including waiting). Image size: 44710204 bytes.
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-5462v       Successfully pulled image "nginx:1.14.2" in 140ms (5.735s including waiting). Image size: 44710204 bytes.
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-bkl8w       Created container: nginx
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-bkl8w       Successfully pulled image "nginx:1.14.2" in 5.882s (5.882s including waiting). Image size: 44710204 bytes.
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-dldmx       Successfully pulled image "nginx:1.14.2" in 170ms (5.711s including waiting). Image size: 44710204 bytes.
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-dldmx       Created container: nginx
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-6r8ck       Created container: nginx
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-6r8ck       Successfully pulled image "nginx:1.14.2" in 194ms (5.606s including waiting). Image size: 44710204 bytes.
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-5462v       Created container: nginx
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-m4dbn       Created container: nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-5462v       Started container nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-bkl8w       Started container nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-dldmx       Started container nginx
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-m4dbn       Successfully pulled image "nginx:1.14.2" in 205ms (5.798s including waiting). Image size: 44710204 bytes.
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-dh2hp       Created container: nginx
10m                     Normal    Pulled                         Pod/nginx-deployment-bf744486c-sz5l2       Successfully pulled image "nginx:1.14.2" in 295ms (6.009s including waiting). Image size: 44710204 bytes.
10m                     Normal    Created                        Pod/nginx-deployment-bf744486c-sz5l2       Created container: nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-sz5l2       Started container nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-6r8ck       Started container nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-dh2hp       Started container nginx
10m                     Normal    Started                        Pod/nginx-deployment-bf744486c-m4dbn       Started container nginx
7m27s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-sz5l2
7m27s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-dldmx       Stopping container nginx
7m27s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-m4dbn       Stopping container nginx
7m27s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-bkl8w       Stopping container nginx
7m27s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-bkl8w
7m27s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-sz5l2       Stopping container nginx
7m27s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 7 to 3
7m27s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-m4dbn
7m27s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-dldmx
6m27s                   Warning   FailedComputeMetricsReplicas   HorizontalPodAutoscaler/nginx-deployment   invalid metrics (1 invalid out of 1), first error is: failed to get cpu resource metric value: failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-5462v
6m27s                   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-5462v
4m57s (x4 over 6m12s)   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-6r8ck
4m57s (x4 over 6m12s)   Warning   FailedComputeMetricsReplicas   HorizontalPodAutoscaler/nginx-deployment   invalid metrics (1 invalid out of 1), first error is: failed to get cpu resource metric value: failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-6r8ck
4m27s (x7 over 7m12s)   Warning   FailedComputeMetricsReplicas   HorizontalPodAutoscaler/nginx-deployment   invalid metrics (1 invalid out of 1), first error is: failed to get cpu resource metric value: failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-dh2hp
4m12s (x8 over 7m12s)   Warning   FailedGetResourceMetric        HorizontalPodAutoscaler/nginx-deployment   failed to get cpu utilization: missing request for cpu in container nginx of Pod nginx-deployment-bf744486c-dh2hp
3m17s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-bf744486c from 3 to 7
3m17s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-6c78464d7c from 0 to 2
3m17s                   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-dtfw9
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-bf744486c-9p8mh       Successfully assigned default/nginx-deployment-bf744486c-9p8mh to node01
3m16s                   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-6c78464d7c     Created pod: nginx-deployment-6c78464d7c-7n4hn
3m16s (x2 over 3m16s)   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      (combined from similar events): Created pod: nginx-deployment-bf744486c-vrh66
3m16s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled up replica set nginx-deployment-6c78464d7c from 2 to 3
3m16s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 7 to 6
3m16s                   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-6c78464d7c     Created pod: nginx-deployment-6c78464d7c-v5mn4
3m16s                   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-6c78464d7c     Created pod: nginx-deployment-6c78464d7c-fx4tz
3m16s                   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-bf744486c      Created pod: nginx-deployment-bf744486c-9p8mh
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-bf744486c-46hhf       Successfully assigned default/nginx-deployment-bf744486c-46hhf to node01
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-bf744486c-vrh66       Successfully assigned default/nginx-deployment-bf744486c-vrh66 to node01
3m16s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-vrh66
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-6c78464d7c-v5mn4      Successfully assigned default/nginx-deployment-6c78464d7c-v5mn4 to node01
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-bf744486c-dtfw9       Successfully assigned default/nginx-deployment-bf744486c-dtfw9 to node01
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-6c78464d7c-fx4tz      Successfully assigned default/nginx-deployment-6c78464d7c-fx4tz to node01
3m16s                   Normal    Scheduled                      Pod/nginx-deployment-6c78464d7c-7n4hn      Successfully assigned default/nginx-deployment-6c78464d7c-7n4hn to node01
3m14s                   Normal    Pulled                         Pod/nginx-deployment-bf744486c-9p8mh       Container image "nginx:1.14.2" already present on machine
3m14s                   Normal    Pulled                         Pod/nginx-deployment-bf744486c-46hhf       Container image "nginx:1.14.2" already present on machine
3m13s                   Normal    Created                        Pod/nginx-deployment-6c78464d7c-7n4hn      Created container: nginx
3m13s                   Normal    Created                        Pod/nginx-deployment-bf744486c-dtfw9       Created container: nginx
3m13s                   Normal    Pulled                         Pod/nginx-deployment-bf744486c-dtfw9       Container image "nginx:1.14.2" already present on machine
3m13s                   Normal    Created                        Pod/nginx-deployment-6c78464d7c-v5mn4      Created container: nginx
3m13s                   Normal    Pulled                         Pod/nginx-deployment-6c78464d7c-fx4tz      Container image "nginx:1.14.2" already present on machine
3m13s                   Normal    Created                        Pod/nginx-deployment-6c78464d7c-fx4tz      Created container: nginx
3m13s                   Normal    Created                        Pod/nginx-deployment-bf744486c-9p8mh       Created container: nginx
3m13s                   Normal    Pulled                         Pod/nginx-deployment-6c78464d7c-7n4hn      Container image "nginx:1.14.2" already present on machine
3m13s                   Normal    Pulled                         Pod/nginx-deployment-6c78464d7c-v5mn4      Container image "nginx:1.14.2" already present on machine
3m13s                   Normal    Created                        Pod/nginx-deployment-bf744486c-46hhf       Created container: nginx
3m12s                   Normal    Started                        Pod/nginx-deployment-bf744486c-9p8mh       Started container nginx
3m12s                   Normal    Started                        Pod/nginx-deployment-6c78464d7c-v5mn4      Started container nginx
3m12s                   Normal    Started                        Pod/nginx-deployment-6c78464d7c-fx4tz      Started container nginx
3m12s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-9p8mh
3m12s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-46hhf
3m12s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-dtfw9
3m12s (x2 over 7m27s)   Normal    SuccessfulRescale              HorizontalPodAutoscaler/nginx-deployment   New size: 3; reason: Current number of replicas above Spec.MaxReplicas
3m12s                   Normal    Started                        Pod/nginx-deployment-6c78464d7c-7n4hn      Started container nginx
3m12s                   Normal    Started                        Pod/nginx-deployment-bf744486c-dtfw9       Started container nginx
3m12s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-6c78464d7c     Deleted pod: nginx-deployment-6c78464d7c-v5mn4
3m12s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-6c78464d7c     Deleted pod: nginx-deployment-6c78464d7c-7n4hn
3m12s                   Normal    Started                        Pod/nginx-deployment-bf744486c-46hhf       Started container nginx
3m12s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 6 to 3
3m11s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      Deleted pod: nginx-deployment-bf744486c-dh2hp
3m11s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-dh2hp       Stopping container nginx
3m11s                   Normal    Killing                        Pod/nginx-deployment-6c78464d7c-7n4hn      Stopping container nginx
3m11s                   Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-6c78464d7c     Created pod: nginx-deployment-6c78464d7c-rxjjh
3m11s                   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-bf744486c from 3 to 2
3m11s                   Normal    Scheduled                      Pod/nginx-deployment-6c78464d7c-rxjjh      Successfully assigned default/nginx-deployment-6c78464d7c-rxjjh to node01
3m11s                   Normal    Killing                        Pod/nginx-deployment-6c78464d7c-v5mn4      Stopping container nginx
3m10s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-dtfw9       Stopping container nginx
3m10s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-9p8mh       Stopping container nginx
3m10s                   Normal    Killing                        Pod/nginx-deployment-bf744486c-46hhf       Stopping container nginx
3m8s                    Normal    Pulled                         Pod/nginx-deployment-6c78464d7c-rxjjh      Container image "nginx:1.14.2" already present on machine
3m8s                    Normal    Created                        Pod/nginx-deployment-6c78464d7c-rxjjh      Created container: nginx
3m7s                    Normal    Started                        Pod/nginx-deployment-6c78464d7c-rxjjh      Started container nginx
3m6s                    Normal    Scheduled                      Pod/nginx-deployment-6c78464d7c-ncrdf      Successfully assigned default/nginx-deployment-6c78464d7c-ncrdf to node01
3m6s                    Normal    SuccessfulCreate               ReplicaSet/nginx-deployment-6c78464d7c     Created pod: nginx-deployment-6c78464d7c-ncrdf
3m6s                    Normal    Killing                        Pod/nginx-deployment-bf744486c-5462v       Stopping container nginx
3m5s                    Normal    Pulled                         Pod/nginx-deployment-6c78464d7c-ncrdf      Container image "nginx:1.14.2" already present on machine
3m5s                    Normal    Created                        Pod/nginx-deployment-6c78464d7c-ncrdf      Created container: nginx
3m4s                    Normal    Started                        Pod/nginx-deployment-6c78464d7c-ncrdf      Started container nginx
3m4s (x2 over 3m6s)     Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-bf744486c      (combined from similar events): Deleted pod: nginx-deployment-bf744486c-6r8ck
3m4s                    Normal    Killing                        Pod/nginx-deployment-bf744486c-6r8ck       Stopping container nginx
3m4s (x4 over 3m11s)    Normal    ScalingReplicaSet              Deployment/nginx-deployment                (combined from similar events): Scaled down replica set nginx-deployment-bf744486c from 1 to 0
2m27s                   Normal    Killing                        Pod/nginx-deployment-6c78464d7c-rxjjh      Stopping container nginx
2m27s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-6c78464d7c     Deleted pod: nginx-deployment-6c78464d7c-fx4tz
2m27s                   Normal    SuccessfulDelete               ReplicaSet/nginx-deployment-6c78464d7c     Deleted pod: nginx-deployment-6c78464d7c-rxjjh
2m27s (x2 over 3m12s)   Normal    ScalingReplicaSet              Deployment/nginx-deployment                Scaled down replica set nginx-deployment-6c78464d7c from 3 to 1
2m27s                   Normal    Killing                        Pod/nginx-deployment-6c78464d7c-fx4tz      Stopping container nginx
2m27s                   Normal    SuccessfulRescale              HorizontalPodAutoscaler/nginx-deployment   New size: 1; reason: All metrics below target

controlplane ~ ➜  





