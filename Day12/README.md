# Day 12: DaemonSets, Jobs, and CronJobs

### What I accomplished today
Today was all about exploring specialized Kubernetes workloads beyond standard Deployments. I learned how to manage cluster-wide infrastructure using DaemonSets and how to schedule and execute temporary, task-based workloads using Jobs and CronJobs. I successfully deployed an Nginx server across all worker nodes and configured a recurring automated task to print a specific message every 5 minutes.

### The YAML Files & Workload Types
I created two separate YAML configurations to handle these distinct tasks. First, I built a DaemonSet configuration that automatically deployed an Nginx container strictly to the worker nodes, respecting the control-plane's default taints. Second, I built a CronJob using standard Linux CRON syntax (`*/5 * * * *`). The interesting part was structuring the deeply nested YAML hierarchy for the CronJob (CronJob -> Job -> Pod) and setting the `restartPolicy` to `OnFailure`. This ensured that the lightweight `busybox` container would gracefully terminate after completing its shell command, rather than restarting endlessly.

### Essential Commands I Used
* **`kubectl apply -f <filename>`** : I used this to deploy both my DaemonSet and CronJob manifests to the cluster.
* **`kubectl get ds`** : This allowed me to verify the desired and current state of my DaemonSet, ensuring it matched the number of available worker nodes.
* **`kubectl get pods -o wide`** : Essential for seeing exactly which nodes the DaemonSet pods were scheduled on, proving they intentionally skipped the master node.
* **`kubectl describe node <node-name> | grep -i taint`** : I used this to investigate the control-plane node and understand the `NoSchedule` security restriction that prevented the DaemonSet from placing a pod there.
* **`kubectl get cj`** : This command helped me monitor the active CronJob, check its schedule, and track its last execution time.
* **`kubectl get pods -w`** : I used the watch flag (`-w`) to observe the CronJob dynamically spinning up a new Pod exactly at the 5-minute mark, watching it transition rapidly from `Pending` to `Completed`.
* **`kubectl logs <completed-pod-name>`** : Once the cron execution finished, I used this to read the output of the terminated Pod and verify my "40daysofkubernetes" message was printed successfully.

### Troubleshooting & Cool Tools
* **YAML Indentation & Array Formatting**: I ran into YAML parsing errors due to incorrect list formatting (leaving a dash `-` on a blank line) and using plural keys (like `templates` instead of `template`). I also learned that the pod `name` field inside a DaemonSet or Deployment template is completely ignored by Kubernetes, as the system auto-generates dynamic pod names.
* **CRON Schedule Logic**: I initially misunderstood the CRON syntax. I learned that using five asterisks (`* * * * *`) schedules a job every single minute. I fixed this by updating the first value to `*/5` to properly enforce the 5-minute interval requirement.
* **Log Retrieval on Ephemeral Pods**: I learned that trying to pull logs using the CronJob's generic name (`kubectl logs pod/hello`) fails because the system generates unique, dynamic Pod names for every execution. Additionally, because Kubernetes automatically cleans up old completed jobs (Successful Jobs History Limit), I had to wait for a fresh execution and specifically target the newly generated Pod's name to view the correct logs before they were garbage-collected.