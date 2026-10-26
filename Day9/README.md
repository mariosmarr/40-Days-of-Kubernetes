# Day 9: Kubernetes Services - ClusterIP vs NodePort

## What I accomplished today

Today I learned how to expose Pods using Kubernetes **Services**, and I tested the practical difference between **internal** cluster access (**ClusterIP**) and **external** access (**NodePort**). I deployed a simple Nginx application and reached it first from inside the cluster, then from my own machine.

## Cluster Setup (KIND)

To make NodePorts reachable from the host machine, KIND needs extra port mappings defined at **cluster creation time**. Without them, the NodePort exists inside the node container but is not exposed to my computer.

**kind create cluster --config kind.yaml**

## Step-by-Step Execution

### 1. Internal access with ClusterIP

I deployed the myapp Deployment (Nginx, 2 replicas) and exposed it with a ClusterIP Service.

**kubectl apply -f myapp.yaml**

**kubectl run temp-busybox --image=busybox -i --tty --rm -- sh**

**wget CLUSTER-IP**

**Result:** The request succeeds, because the busybox Pod runs inside the cluster network. Running the same wget from my host terminal fails, which proves that ClusterIP is strictly internal.

### 2. External access with NodePort

To allow outside traffic, I changed the Service type from ClusterIP to NodePort.

**kubectl apply -f myapp.yaml**

**kubectl get svc myapp**

**Result:** Running wget NODE-IP:NODE-PORT from outside the cluster now returns the Nginx landing page, confirming external reachability.

## Essential Commands I Used

**kind create cluster --config kind.yaml** creates the local cluster with the port mappings needed for NodePort.

**kubectl apply -f myapp.yaml** creates or updates the resources defined in the manifest.

**kubectl run temp-busybox --image=busybox -i --tty --rm -- sh** starts a throwaway Pod with an interactive shell, deleted automatically on exit.

**kubectl get svc myapp** shows the Service details, including its type, ClusterIP and NodePort.

## Theory & Key Takeaways

### Can you expose Pods as a Service without a Deployment?

**Yes.** A Service routes traffic based purely on Labels and Selectors. It doesn't care whether a Pod was created by a Deployment, a ReplicaSet, or manually as a standalone Pod. As long as the Pod's labels match the Service's selector, traffic is routed to it.

### When to use each Service type

**ClusterIP** is the default type. It is used for internal communication between application components, for example a Backend talking to a Database. No external access.

**NodePort** opens a specific port (30000-32767) on all Worker Nodes. It is mainly used for local development, testing, or on-premise setups without cloud integration.

**LoadBalancer** is used in production on cloud providers (AWS, GCP, Azure). It provisions a public-facing IP and routes external internet traffic into the cluster.

**ExternalName** acts as a DNS CNAME record. It lets Pods reach an external service (outside the cluster) using a local Kubernetes DNS name.