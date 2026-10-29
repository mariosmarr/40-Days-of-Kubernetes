# Day 11: Environment Variables and Multi-Container Pods

### What I accomplished today
Today was all about diving deeper into Kubernetes Pod architecture by creating a multi-container Pod. I learned how to pass internal environment variables to my application and, most importantly, how to use `initContainers` as "gatekeepers." These gatekeepers run sequentially and block the main application container from starting until specific network dependencies are fully available and reachable.

### The YAML File & Gatekeepers
I created a Pod configuration containing one main container and two `initContainers`. The interesting part was setting up the startup sequence. The main container was injected with custom environment variables (like `FIRSTNAME`), but it remained in a `PodInitializing` state. The `initContainers` executed a shell loop using `nslookup`, constantly querying the Kubernetes DNS for specific Services. Only when those Services were manually created and detected did the init containers terminate successfully, passing the baton to the main application.

### Essential Commands I Used
* **`kubectl apply -f <filename>`** : I used this to deploy my multi-container Pod from the YAML manifest.
* **`kubectl create deploy <name> --image=<image> --port=<port>`** : This allowed me to quickly spin up a dummy backend application from the terminal to satisfy the init container's requirements.
* **`kubectl expose deploy <name> --name=<service-name> --port=<port>`** : I used this to expose the deployment and create the exact Service name the init container was waiting for, effectively "unlocking" the Pod.
* **`kubectl get pods`** : Essential for monitoring the exact status of the Pod, watching it transition from `Init:0/2` to `1/1 Running`.
* **`kubectl logs <pod-name> -c <container-name>`** : Since the Pod had multiple containers, I learned to use the `-c` flag to specifically target and read the logs of the stuck init containers.
* **`kubectl exec -it <pod-name> -- printenv`** : Once the Pod was running, I used this to execute a command inside the live container to verify that my environment variables were successfully passed.

### Troubleshooting & Cool Tools
* **YAML Syntax & Loop Formatting**: I ran into YAML parsing errors due to unclosed brackets in my arrays. I also learned that while `initContainers` need a complete shell loop structure , the main container needs a linear command , otherwise it crashes.
* **The `nslookup` DNS Fix**: My init container loop was initially failing because it tried to run the service URL as a standalone command. I fixed this by adding `nslookup` to properly query the network.
* **Fully Qualified Domain Names (FQDN)**: The init containers couldn't find the Services using just their short names. I resolved this by updating the YAML to search for the full internal Kubernetes address, explicitly adding the namespace: `myservice.default.svc.cluster.local`.