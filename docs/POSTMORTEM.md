# Blameless Incident Postmortem
**Incident Date:** Current Evaluation Date  
**Author / Responder:** DevOps Engineering Team  
**Status:** Resolved  

---

## Incident Summary & Impact
During the Stan's Robot Shop deployment and validation phase, the Redis stateful data store experienced persistent volume binding failures. This caused dependent services (such as the `cart` and checkout workflows) to experience connection timeouts, interrupting user traffic simulation via `load-gen`.

## Timeline (UTC)
* **14:00** – Robot Shop helm chart deployed; pods initialized.
* **14:05** – Redis pod reported `Pending` state due to persistent volume claim (PVC) binding deadlock.
* **14:12** – Investigation initiated via `kubectl get pods` and Prometheus metrics monitoring.
* **14:20** – Mitigation applied: Bypassed the cloud provider storage class lock by converting Redis to a standard Deployment using ephemeral `emptyDir` storage.
* **14:25** – Pods restarted, verified healthy (`1/1 Running`), and traffic resumed successfully.

## Root Cause Analysis
The persistent volume provisioner failed to dynamically bind the cloud storage volume to the Redis pod within the Kubernetes cluster, leading to a startup deadlock. State management for caching did not require persistent EBS storage for the scope of this non-production workload environment.

## Corrective Actions & Preventive Measures
1. **Infrastructure Adaptation:** Updated the Redis deployment configuration to utilize local ephemeral storage (`emptyDir`) for testing stability.
2. **Runbook Documentation:** Added explicit storage troubleshooting steps to the team runbook for handling PVC binding deadlocks.
3. **Monitoring Enhancement:** Configured Prometheus metric alerts to notify engineers immediately on pod pending timeouts.
