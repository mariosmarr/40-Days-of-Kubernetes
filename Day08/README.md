# Day 8: ReplicaSets, Deployments, and Rollbacks

### What I accomplished today
Today was all about moving beyond single Pods and learning how to manage applications at scale. I worked with ReplicaSets to ensure a specific number of Pods are always running, and then moved on to Deployments, which are the standard way to deploy applications in Kubernetes. The highlight was performing a zero-downtime rolling update and then successfully rolling it back to a previous working state using the rollout history.

### Key Concepts & Troubleshooting
- **API Versions & Indentation**: I fixed broken YAML manifests by correcting the `apiVersion` for Deployments (it must be `apps/v1`, not `v1`) and fixing indentation errors under the `spec` section.
- **Label Matching Rule**: I learned that the `selector.matchLabels` must exactly match the `template.metadata.labels` of the Pod. If they don't match, the Deployment won't work.
- **Immutable Fields**: I discovered that once a Deployment is created, its label selector (`matchLabels`) is immutable. To change it, you have to completely delete and recreate the Deployment.

### Essential Commands I Used
- **`kubectl create deployment nginx --image=nginx:1.23.0 --dry-run=client -o yaml > deploy.yaml`**: An incredible shortcut (Imperative Generator) that automatically creates the YAML skeleton for a Deployment so I don't have to write it from scratch.
- **`kubectl scale deployment nginx --replicas=5`**: This instantly scales the number of running Pods up or down directly from the command line, without editing the YAML file.
- **`kubectl annotate deployment nginx kubernetes.io/change-cause="<message>"`**: This attaches a note to the current deployment revision, making it easy to track *why* an update was made in the history.
- **`kubectl rollout history deployment/nginx`**: Displays the "logbook" of all the revisions/updates made to the deployment.
- **`kubectl rollout undo deployment/nginx --to-revision=1`**: The lifesaver command! It instantly reverts the deployment back to a previous state (Revision 1) if something goes wrong with the new update.