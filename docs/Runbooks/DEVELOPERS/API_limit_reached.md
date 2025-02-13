# API Product Runbook

## Overview
**Service Name:** API Product Service  
**Severity:** Varies based on issue  
**Impact:** Affected API endpoints may cause degraded performance or outages for clients.

## Common Alerts
### 1. API Latency High
- **Alert Condition:** API response time exceeds threshold (e.g., >500ms).
- **Troubleshooting Steps:**
  1. Check API logs for slow queries or timeouts:
     ```sh
     kubectl logs -l app=api-product -n <namespace> --tail=100
     ```
  2. Monitor database performance:
     ```sh
     kubectl exec -it <db-pod> -- psql -U user -d database -c "SELECT * FROM pg_stat_activity;"
     ```
  3. Analyze recent deployments for performance regressions.
  4. Scale API pods if needed:
     ```sh
     kubectl scale deployment api-product --replicas=<new_count>
     ```

### 2. API Error Rate High
- **Alert Condition:** HTTP 5xx errors exceed threshold.
- **Troubleshooting Steps:**
  1. Identify failing endpoints from logs:
     ```sh
     kubectl logs -l app=api-product -n <namespace> | grep "5xx"
     ```
  2. Check API Gateway logs for upstream errors.
  3. Verify database connectivity:
     ```sh
     kubectl exec -it <db-pod> -- psql -U user -d database -c "SELECT pg_is_in_recovery();"
     ```
  4. Restart affected pods:
     ```sh
     kubectl rollout restart deployment api-product
     ```

### 3. Authentication Failures
- **Alert Condition:** Increase in failed authentication requests.
- **Troubleshooting Steps:**
  1. Check logs for authentication errors:
     ```sh
     kubectl logs -l app=api-product -n <namespace> | grep "401"
     ```
  2. Verify authentication service is healthy.
  3. Check token expiration and user credentials.

## Remediation Steps
- **Optimize API queries:** Use indexing and caching where applicable.
- **Implement circuit breakers:** Use retry and failover mechanisms.
- **Monitor dependencies:** Ensure external services (DB, Auth) are available.

## Escalation
- **SRE Team Slack:** `#api-alerts`
- **On-Call Engineer:** `PagerDuty rotation`
- **Product Team Contact:** `api-team@example.com`

## References
- API Logging Best Practices: [GitHub Docs](https://docs.github.com/en/rest)
- Kubernetes Monitoring: [Kubernetes Docs](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
- Database Performance Tuning: [PostgreSQL Docs](https://www.postgresql.org/docs/)
