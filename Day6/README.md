# Day 6: Setting up a Local Kubernetes Cluster with KIND

### What I accomplished today
Today was all about moving from Kubernetes theory to hands-on practice. My goal was to build a local Kubernetes lab on my machine without needing to provision cloud servers. I learned how **KIND** (Kubernetes IN Docker) works, which efficiently uses Docker containers to simulate real Kubernetes nodes. I also installed and configured the **kubectl** client to interact directly with my local clusters.

### The Cluster Setup
I started by creating a simple single-node cluster (v1.29) to understand the basic workflow, and then deleted it to free up local resources. Next, I deployed a full **multi-node cluster** (named `cka-cluster2` running Kubernetes v1.30) containing 1 Control Plane and 3 Worker nodes using a custom `config.yaml` file. 

### Essential Commands I Used

- `kind create cluster --image kindest/node:v1.30.0` : Created a baseline single-node cluster.
- `kind delete cluster` : Cleaned up the local resources by deleting the initial cluster.
- `kind create cluster --name cka-cluster2 --image kindest/node:v1.30.0 --config config.yaml` : Bootstrapped the multi-node production-like cluster using my custom configuration.
- `kubectl config use-context kind-cka-cluster2` : Pointed my `kubectl` client to the newly created multi-node cluster.
- `kubectl get nodes` : Verified that all 4 nodes (1 control-plane, 3 workers) were successfully provisioned and in "Ready" status.
- `docker ps` : Proved the architectural concept of KIND by verifying that the 4 Kubernetes nodes are actually running as 4 isolated Docker containers on my machine.
