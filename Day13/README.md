# Day 13: Static Pods, Manual Scheduling, and Labels



## What I accomplished today



Today I explored the core mechanics of Kubernetes scheduling and resource organization. I learned how to bypass the central scheduler to manually assign Pods to specific nodes, discovered that Control Plane components run as **Static Pods** managed by the **Kubelet**, and used **Labels** and **Selectors** to logically group and filter my workloads.



## Cluster Setup (KIND)



For this exercise, I used my existing local KIND cluster. The preparation step was verifying the available nodes, so I could manually target a specific worker node for my workloads.



**kubectl get nodes**



## Step-by-Step Execution



### 1. Manual Pod Scheduling



I created an Nginx Pod manifest and explicitly assigned it to a worker node by adding the `nodeName` field directly into the Pod specification. Because the node is already defined, the **kube-scheduler** is never involved in the decision, and the Pod is run directly by the **Kubelet** on that node.



**kubectl apply -f manual-pod.yaml**



### 2. Managing Static Pods and the Control Plane



I accessed the control plane node directly through Docker to inspect the default static pod manifests directory. Static Pods are not created through the API server. The Kubelet watches this directory and runs whatever manifests it finds there, which is how components like the scheduler are started. By moving the scheduler manifest out of the directory, the Kubelet stopped the Kubernetes Scheduler. By moving it back, the Kubelet recreated it automatically, without any interaction with the API server.



**docker exec -it cka-cluster3-control-plane bash**



**cd /etc/kubernetes/manifests**



**mv kube-scheduler.yaml /tmp**



**mv /tmp/kube-scheduler.yaml .**



### 3. Organizing Workloads with Labels



I created three separate Nginx Pods and assigned each one a unique `env` label with the values `test`, `dev`, and `prod`. After deploying them, I used **label selectors** in my kubectl commands to filter the cluster view and query only the specific environments I needed.



**kubectl apply -f task.pod1.yaml**



**kubectl apply -f task.pod2.yaml**



**kubectl apply -f task.pod3.yaml**



**kubectl get pods -l env=prod**



**kubectl get pods -l env=test**



## Key Takeaways



**Manual scheduling** with `nodeName` skips the scheduler entirely and places the Pod directly on the chosen node.



**Static Pods** are managed by the Kubelet from a local directory, which is why the control plane can start even before the API server is running.



**Labels and Selectors** are the main way Kubernetes groups and filters resources, and they are the foundation for objects such as Services and Deployments.