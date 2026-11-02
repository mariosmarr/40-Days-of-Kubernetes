# Day 16: Kubernetes Resource Management (Requests & Limits)



## What I accomplished today



Today, I explored one of the most critical aspects of cluster stability: **Resource Management**. I installed the **Metrics Server** to monitor real-time consumption and experimented with CPU and Memory `requests` and `limits`. By intentionally misconfiguring Pods, I practically observed how the Kubernetes Scheduler and Kubelet protect the nodes from resource starvation and noisy neighbors.





## Cluster Setup & Prerequisites



Before deploying any resource-bound Pods, I created a dedicated namespace to isolate my tests.



**kubectl create namespace mem-example**



I also deployed the **Metrics Server** in the `kube-system` namespace. This add-on is mandatory for tracking actual resource utilization (enabling the `kubectl top` command) and is the backbone for **Horizontal Pod Autoscaling**.



## Step-by-Step Execution & Scenarios



### 1. The "Happy Path" (Healthy Execution)



I deployed a Pod (`memory-demo`) containing a `stress` container configured to consume exactly 150Mi of RAM. The Request was set to 100Mi and the Limit to 200Mi.



**Result:** The Pod ran successfully because 150Mi is above the guaranteed baseline but safely below the absolute limit. `kubectl top pod` confirmed the ~150Mi usage.



### 2. Exceeding the Limit (OOMKilled)



I deployed a second Pod (`memory-demo-2`) with a strict Limit of 100Mi, but instructed the application to allocate 250Mi.



**Result:** The Pod status immediately changed to `OOMKilled` (Exit Code 137), followed by a `CrashLoopBackOff`. This proved that RAM is an **incompressible resource**. If a container breaches its memory limit, the Kubelet terminates it instantly to protect the Node.



### 3. Exceeding Node Capacity (Pending)



I deployed a third Pod (`memory-demo-3`) requesting an absurd amount of memory (1000Gi Request).



**Result:** The Pod never started. It remained in a `Pending` state. The `kubectl describe pod` command revealed a `FailedScheduling` warning due to `Insufficient memory`. This demonstrated that the Scheduler looks at **Requests** (not Limits) to place Pods, and will block deployment if no Node can guarantee the requested baseline.



## Essential Commands I Used



**kubectl top nodes / kubectl top pods** displays real-time CPU and Memory utilization (requires Metrics Server).



**kubectl describe pod POD_NAME** reveals the lifecycle events of a Pod, crucial for debugging `FailedScheduling` or `CrashLoopBackOff` issues.



**kubectl delete namespace mem-example** safely and instantly cleans up all resources, Pods, and configurations created during the experiment.



## Theory & Key Takeaways



### Requests vs. Limits



**Requests (The Guarantee):** What the Pod needs to start. The Scheduler uses this number to find a suitable Node. If the requested resources aren't available cluster-wide, the Pod stays `Pending`.



**Limits (The Ceiling):** The absolute maximum the Pod is allowed to consume. This protects the Node from memory leaks or traffic spikes in a single container.



### Compressible vs. Incompressible Resources



**CPU is compressible:** If a Pod exceeds its CPU limit, it is simply throttled (slowed down). It is not terminated.



**Memory is incompressible:** If a Pod exceeds its Memory limit, there is no throttling. The system invokes the **OOM (Out Of Memory) Killer**, terminating the container immediately to prevent Node crashes.