# Day 15: Kubernetes Node Affinity



## What I accomplished today



Today I explored **Node Affinity**, the advanced scheduling mechanism in Kubernetes that attracts Pods to specific nodes. Unlike **Taints** (which repel Pods), Node Affinity allows Pods to define strict or preferred rules about where they want to run based on node labels. I tested the strict (`required`) affinity rules to control the exact placement of Nginx and Redis Pods.



## Cluster Setup (KIND)



For this exercise, I used my local KIND cluster consisting of one Control Plane and two Worker nodes. I started with a clean slate, ensuring no custom labels or taints were interfering with the scheduling process.



## Step-by-Step Execution



### 1. Hard Affinity & Pending State



I created a YAML manifest for an Nginx Pod, injecting a `requiredDuringSchedulingIgnoredDuringExecution` affinity rule. The rule explicitly demanded a node with the label `disktype=ssd`.



**kubectl apply -f affinity-task.yaml**



**kubectl get pods**



**Result:** The Pod remained in a `Pending` state. The Scheduler events confirmed `FailedScheduling` because 0/3 nodes matched the Pod's node affinity selector.



### 2. Satisfying the Affinity Rule



To resolve the pending state, I applied the exact label the Pod was looking for to my first worker node.



**kubectl label node cka-cluster3-worker disktype=ssd**



**kubectl get pods**



**Result:** The Scheduler immediately detected the label update, matched the affinity rule, and transitioned the Nginx Pod to a `Running` state on `cka-cluster3-worker`.



### 3. Key-Only Affinity (Without Value)



I created a second Pod (Redis) with a required node affinity, but this time using the `Exists` operator (or an empty value string). This rule demanded a node that simply possessed the `disktype` key, regardless of its actual value.



**kubectl apply -f affinity-task2.yaml**



**kubectl label node cka-cluster3-worker2 disktype=**



**Result:** The Pod was successfully scheduled. **Note on Kubernetes behavior:** Because `cka-cluster3-worker` already had the label `disktype=ssd`, it also satisfied the "key exists" condition. This highlights how the Scheduler evaluates overlapping labels when using broad affinity rules.



## Essential Commands I Used



**kubectl label node NODE-NAME key=value** adds a new label to a specific node.



**kubectl label node NODE-NAME key-** removes a label from a node (by appending a minus sign).



**kubectl get nodes --show-labels** lists all nodes along with their currently attached labels.



## Theory & Key Takeaways



### Why use Node Affinity over NodeSelector?



While `nodeSelector` is a simple way to assign Pods to nodes, **Node Affinity** provides a vastly more expressive syntax. It supports logical operators like `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, and `Lt`, allowing for complex scheduling logic (e.g., "Schedule on a node in Zone A OR Zone B").



### Hard vs. Soft Affinity



**Required (Hard):** `requiredDuringSchedulingIgnoredDuringExecution`. The Scheduler will absolutely not place the Pod on a node unless the rule is perfectly met. If no node matches, the Pod stays `Pending`.



**Preferred (Soft):** `preferredDuringSchedulingIgnoredDuringExecution`. The Scheduler treats the rule as a wish. It will try its best to place the Pod on a matching node, but if none are available, it will gracefully schedule the Pod on any standard available node to ensure the application stays online.