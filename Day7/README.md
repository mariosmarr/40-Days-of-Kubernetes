# Day 7: Imperative vs. Declarative & YAML Troubleshooting

### What I accomplished today
Today was all about shifting from imperative commands to a declarative approach. I learned how to generate YAML templates dynamically and deploy pods using Infrastructure as Code. I also practiced troubleshooting broken YAML configurations.

### Essential Commands I Used
* **`kubectl run nginx --image=nginx`** : Created a pod instantly using the imperative approach.
* **`kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-new.yaml`** : Generated a YAML template without actually creating the pod, saving time and preventing syntax typos.
* **`kubectl create -f nginx-new.yaml`** : Deployed a new pod using the newly generated declarative YAML file.
* **`kubectl describe pod redis`** : Inspected the pod's events to find the exact reason it failed to start.
* **`kubectl apply -f redis-broken.yaml`** : Applied the corrected YAML configuration to the cluster.

### Troubleshooting & Fixes
* **Fixing YAML Syntax**: The provided `redis-broken.yaml` file had flattened key-value pairs. I reformatted the code with the correct spacing and indentation to meet Kubernetes' strict YAML standards.
* **Debugging `ErrImagePull`**: After applying the YAML, the pod failed to start and threw an `ImagePullBackOff` error. Using the `describe` command, I found the root cause: `Failed to pull image "rediss"`. The image name was intentionally misspelled. I corrected it to `image: redis`, reapplied the file, and the pod successfully transitioned to the `Running` state.