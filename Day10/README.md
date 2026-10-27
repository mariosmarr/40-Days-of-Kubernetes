# Day 10: Kubernetes Namespaces and DNS Resolution (FQDN)

## What I accomplished today

Today I learned how to organize Kubernetes resources using **Namespaces**, and I tested how Pods communicate across different namespaces. I deployed Nginx applications in separate namespaces, scaled them, and proved that while **IP communication** works directly, **name-based communication (DNS)** requires the use of an **FQDN** (Fully Qualified Domain Name).

## Cluster Setup (KIND)

For this exercise, I used my existing local KIND cluster. The main preparation step was creating two separate logical boundaries (namespaces) in my cluster to isolate my resources.

**kubectl create namespace ns1**

**kubectl create namespace ns2**

## Step-by-Step Execution

### 1. Pod-to-Pod communication via IP

I created an Nginx Deployment in each namespace (`ns1` and `ns2`) with 1 replica each. Then, I found the IP address of the Pod in `ns2` and successfully reached it from inside the Pod in `ns1`.

**kubectl create deployment nginx-ns1 --image=nginx --replicas=1 -n ns1**

**kubectl create deployment nginx-ns2 --image=nginx --replicas=1 -n ns2**

**kubectl get pods -n ns2 -o wide**

**kubectl exec -it POD-NAME -n ns1 -- sh**

**curl POD-IP-NS2**

**Result:** The request succeeds, returning the Nginx landing page. This proves that Namespaces are **not physical network firewalls**; Pods can communicate directly via IP regardless of the namespace they belong to.

### 2. Service communication & DNS Resolution

I scaled the Deployments to 3 replicas and created two ClusterIP Services (`svc-ns1` and `svc-ns2`). I then entered the Pod in `ns1` again to test communication using the Service name instead of the IP.

**kubectl scale deploy/nginx-ns1 --replicas=3 -n ns1**

**kubectl scale deploy/nginx-ns2 --replicas=3 -n ns2**

**kubectl expose deployment/nginx-ns1 --name=svc-ns1 --port=80 -n ns1**

**kubectl expose deployment/nginx-ns2 --name=svc-ns2 --port=80 -n ns2**

**curl svc-ns2**

**curl svc-ns2.ns2.svc.cluster.local**

**Result:** The first command (`curl svc-ns2`) **fails** (*Could not resolve host*) because the system searches for the service only in the local namespace (`ns1`). The second command using the full name (`curl svc-ns2.ns2.svc.cluster.local`) **succeeds immediately**, demonstrating how DNS resolution works in Kubernetes.

## Essential Commands I Used

**kubectl create namespace ns1** creates a new isolated logical boundary (namespace) in the cluster.

**kubectl get pods -n ns1 -o wide** lists the Pods in a specific namespace, displaying extra information like their internal IP addresses.

**kubectl exec -it POD-NAME -n ns1 -- sh** opens an interactive terminal connection inside a specific Pod.

**kubectl scale deploy/DEPLOYMENT-NAME --replicas=3 -n ns1** increases or decreases the number of Pods (replicas) in a Deployment.

**kubectl delete namespace ns1 ns2** deletes the namespaces, automatically and safely destroying all resources (Deployments, Pods, Services) inside them.

## Theory & Key Takeaways

### What exactly are Namespaces?

Namespaces are **logical organizational folders**, not physical hardware separations (they are not separate Worker Nodes). They help isolate environments (e.g., Dev, Test, Prod), prevent name collisions, and control access permissions. They **do not block network traffic by default**.

### Why does communication fail using the short Service name?

Every Pod is internally configured (in its `/etc/resolv.conf` file) to search for services **first in its own Namespace**. If a Pod in `ns1` requests `svc-ns2` using its short name, the internal DNS (**CoreDNS**) looks for it exclusively inside `ns1`. Since it's not there, it returns a "Could not resolve host" error.

### What is FQDN and how does it solve the problem?

To reach a service in a different Namespace by name, we must use the **FQDN** (Fully Qualified Domain Name), which is the complete internal network address in Kubernetes. Its strict structure is:

**SERVICE-NAME.NAMESPACE.svc.cluster.local**

By providing this full address, we bypass the local search, and CoreDNS knows exactly which namespace to route the request to.