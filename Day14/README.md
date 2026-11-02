# Day 14: Kubernetes Taints and Tolerations



## What I accomplished today



Today I learned how to control Pod scheduling and protect specific nodes using **Taints** and **Tolerations**. I tested how applying Taints to worker nodes prevents standard Pods from being scheduled. I then deployed Pods using specific Tolerations to bypass these restrictions, and I successfully manipulated the default Taints on the Control Plane to temporarily schedule a Redis workload there.



## Cluster Setup (KIND)



For this exercise, I used my existing local KIND cluster. The main preparation step was applying specific restrictions (Taints) to both of my worker nodes to prevent any regular Pods from being scheduled on them.



**kubectl taint node cka-cluster3-worker gpu=true:NoSchedule**



**kubectl taint node cka-cluster3-worker2 gpu=false:NoSchedule**



## Step-by-Step Execution



### 1. Testing Default Scheduling (Pending State)



I created a standard Nginx Pod without configuring any special permissions (Tolerations). Because all worker nodes were restricted by Taints and the Control Plane does not accept normal workloads by default, the Pod had nowhere to go and remained stuck in a `Pending` state.



**kubectl run nginx --image=nginx**



**kubectl get pods**



**Result:** The Pod status shows as `Pending`. Checking the Pod events reveals a `FailedScheduling` warning because the nodes had untolerated taints.



### 2. Adding a Toleration to a Pod



I deleted the stuck Nginx Pod and created a new Pod manifest that included a specific Toleration matching the exact Taint on my first worker node. Once applied, the Pod successfully bypassed the restriction and was scheduled on the targeted worker node.



**kubectl apply -f task.pod.yaml**



**kubectl get pods**



**Result:** The Pod status changes to `Running`. The Toleration acts as a VIP pass, allowing the Pod to be placed on the tainted worker node.



### 3. Scheduling on the Control Plane



To test Control Plane scheduling rules, I temporarily removed its default protective Taint. I then deployed a Redis Pod, which successfully scheduled directly on the Control Plane node. Afterward, I restored the original Taint to secure the Control Plane.



**kubectl taint node cka-cluster3-control-plane node-role.kubernetes.io/control-plane:NoSchedule-**



**kubectl run redis --image=redis**



**kubectl taint node cka-cluster3-control-plane node-role.kubernetes.io/control-plane:NoSchedule**



**Result:** The Redis Pod transitions to `Running` on the Control Plane node, proving that the default Taint is the only mechanism preventing regular workloads from running there.



## Essential Commands I Used



**kubectl taint node NODE-NAME key=value:Effect** applies a restriction (Taint) to a specific node to repel Pods.



**kubectl taint node NODE-NAME key=value:Effect-** removes an existing Taint from a node by adding a minus sign at the end of the command.



**kubectl describe node NODE-NAME | grep Taints** checks the current Taints applied to a specific node.



**kubectl run POD-NAME --image=IMAGE-NAME --dry-run=client -o yaml > file.yaml** generates a YAML template for a Pod without actually creating it, allowing me to manually add configurations like Tolerations.



## Theory & Key Takeaways



### What exactly are Taints?



**Taints** are node-level restrictions acting as a repellent. They do not dictate where a Pod must go; instead, they dictate where a Pod is not allowed to go. They are primarily used to reserve specific nodes for special workloads (like GPU-intensive tasks) or to protect core infrastructure (like the Control Plane).



### What exactly are Tolerations?



**Tolerations** are Pod-level permissions acting as VIP passes. If a node has a Taint, a Pod can only be scheduled on it if its YAML manifest contains a Toleration that perfectly matches the node's Taint key, value, and effect.



### What is the NoSchedule effect?



The **NoSchedule** effect is a strict rule that prevents any new Pods from being placed on the tainted node unless they have the matching Toleration. It does not evict Pods that were already running on the node before the Taint was applied.