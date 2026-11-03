# Day 17: Kubernetes Autoscaling (HPA vs VPA)

## What I accomplished today

Today I explored how Kubernetes dynamically handles traffic spikes and resource demands through **Autoscaling**. I learned the fundamental differences between Horizontal and Vertical scaling and successfully implemented an HPA (Horizontal Pod Autoscaler) rule via the CLI. I then simulated a real-world high-traffic event (Load Generation) to observe the HPA automatically scale up my application to maintain a target CPU utilization, and subsequently scale it back down when the load ceased.

## The Three Pillars of Kubernetes Autoscaling

### 1. Horizontal Pod Autoscaler (HPA) - Scaling Out/In
HPA solves traffic spikes by adjusting the **number of Pod replicas**. If an application is overwhelmed, HPA deploys additional identical Pods to distribute the incoming traffic (Load Balancing). When the traffic subsides, it terminates the unneeded replicas to save cluster resources. 
*   *Requirement:* HPA strictly requires the Metrics Server to be installed and `resources.requests` to be defined in the Pod's YAML manifest.

### 2. Vertical Pod Autoscaler (VPA) - Scaling Up/Down
VPA solves resource constraints by adjusting the **size (CPU/RAM) of existing Pods**. If a container frequently hits its memory limit (risking an OOMKilled status), VPA intervenes by gracefully restarting the Pod with higher `requests` and `limits`. It replaces a weak container with a more powerful one, rather than adding more containers.

### 3. Cluster Autoscaler (CA) - Infrastructure Scaling
While HPA and VPA manage Pods, the Cluster Autoscaler manages the underlying infrastructure. If HPA requests more Pods than the physical Worker Nodes can accommodate (resulting in `Pending` Pods), the CA communicates with the Cloud Provider to provision entirely new VMs/Nodes.

## Hands-on Experiment: HPA in Action

To see HPA in action, I deployed a sample PHP-Apache application with a CPU request of `200m`.

### 1. Creating the HPA Rule
I generated an HPA controller using an imperative command, instructing Kubernetes to maintain an average CPU utilization of 50%, allowing it to scale between 1 and 10 replicas:
**kubectl autoscale deploy php-apache --cpu-percent=50 --min=1 --max=10**

### 2. Simulating High Load (The Attack)
I opened a secondary terminal and deployed a temporary busybox pod to spam the Apache service with requests every 0.01 seconds:
**kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"**

### 3. Observing the Equilibrium
By monitoring the HPA with **kubectl get hpa -w**, I watched the average CPU utilization spike to over 200%. The HPA immediately reacted by calculating the required replicas to bring the average back to the 50% target. Within moments, the replicas scaled up to **7 Pods**, successfully dropping the CPU average per Pod to ~47-50%. The system reached perfect equilibrium. 

### 4. Cooldown and Scale Down
Once I terminated the load generator, the CPU utilization dropped to 0%. The HPA waited for the default cooldown period (approx. 5 minutes) to ensure the traffic drop wasn't a temporary anomaly, and then gracefully terminated 6 Pods, scaling back down to the minimum of 1 replica.

## Essential Commands I Used

**kubectl autoscale deploy DEPLOYMENT-NAME --cpu-percent=50 --min=1 --max=10** automatically scales a Deployment based on observed CPU utilization.

**kubectl get hpa -w** watches the status of the Horizontal Pod Autoscaler in real-time.

**kubectl run -i --tty POD-NAME --image=busybox -- /bin/sh** creates and runs a temporary interactive pod.

**kubectl delete hpa HPA-NAME** deletes a specific Horizontal Pod Autoscaler rule.

## Key Takeaway

Memory caching behaviors (where applications hold onto RAM even when idle) can trick an autoscaler into believing a system is still under heavy load. Therefore, **CPU utilization** is the gold standard metric for triggering Horizontal Pod Autoscaling in production environments.