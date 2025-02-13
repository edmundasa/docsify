# Kubernetes OOM Alert Runbook

## Overview
**Alert Name:** Kubernetes OOM (Out of Memory) Alert  
**Severity:** High  
**Impact:** Applications may crash or become unresponsive due to insufficient memory.

## Symptoms
- Pod enters `OOMKilled` state.
- Application logs show `OOMKilled` events.
- Node memory usage spikes close to 100%.
- Prometheus alert: `container_memory_usage_bytes` exceeding limits.

## Immediate Actions
1. **Identify the affected pod(s):**  
   ```sh
   kubectl get pods --all-namespaces | grep OOMKilled
   ```
2. **Check pod logs for errors:**  
   ```sh
   kubectl logs <pod-name> -n <namespace>
   ```
3. **Check events for the pod:**  
   ```sh
   kubectl describe pod <pod-name> -n <namespace>
   ```
4. **Check node resource usage:**  
   ```sh
   kubectl top nodes
   kubectl top pods --namespace=<namespace>
   ```

## Root Cause Analysis (RCA)
### Common Causes
- **Memory limits too low** for the container.
- **Memory leaks** in the application.
- **Node memory exhaustion** due to high usage by other workloads.
- **Unexpected traffic spikes** causing high memory consumption.

### Debugging Steps
1. **Verify resource limits:**
   ```sh
   kubectl get pod <pod-name> -o yaml | grep -A5 'resources:'
   ```
2. **Analyze memory usage trends:**
   - Use Prometheus/Grafana dashboards.
   - Run `kubectl top pods` and `kubectl top nodes`.
3. **Check kernel logs for OOM events:**
   ```sh
   dmesg | grep -i 'oom'
   ```
4. **Inspect application logs:**
   - Look for `OutOfMemoryError` (Java), `std::bad_alloc` (C++), or similar messages.

## Remediation Steps
### Temporary Fix
- **Restart the affected pod:**
  ```sh
  kubectl delete pod <pod-name> -n <namespace>
  ```
- **Scale up replicas:**
  ```sh
  kubectl scale deployment <deployment-name> --replicas=<new_count>
  ```

### Permanent Fix
- **Increase memory requests/limits in the deployment YAML:**
  ```yaml
  resources:
    requests:
      memory: "256Mi"
    limits:
      memory: "512Mi"
  ```
- **Optimize application memory usage:**
  - Tune JVM heap settings for Java apps.
  - Fix memory leaks in the application code.
- **Use Horizontal Pod Autoscaler (HPA):**
  ```sh
  kubectl autoscale deployment <deployment-name> --cpu-percent=70 --min=2 --max=10
  ```

## Validation
- Ensure pods are running normally:
  ```sh
  kubectl get pods -n <namespace>
  ```
- Verify memory usage has stabilized using monitoring tools.
- Check logs for any new OOM errors.

## Escalation
If the issue persists, escalate to:
- **SRE Team Slack:** `#sre-alerts`
- **On-Call Engineer:** `PagerDuty rotation`
- **Platform Team Email:** `platform-team@example.com`

## References
- Kubernetes Resource Management: [Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/manage-resources/)
- Troubleshooting OOM Issues: [GitLab Docs](https://docs.gitlab.com/ee/administration/troubleshooting/kubernetes_oom.html)
- Prometheus Alerting: [GitHub Docs](https://prometheus.io/docs/alerting/latest/alerts/)