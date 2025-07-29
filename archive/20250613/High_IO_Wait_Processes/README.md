## Notebook

**SRE Investigation: Correlated Process Metrics for High IO Wait Alerts**

### Markdown Widget

#### High IO Wait correlation Events & Processes by Resource Consumption

#### High IO Wait Alert Metrics Within 36h Process Retention Window



#### Monitor Details

**Production - High IO wait on k8s worker node**
- Query
`avg(last_30m):avg:system.cpu.iowait{env:*prod*} by {instance,host,env} > 25`
  - Critical: Avg iowait in the last 30 minutes > 25%
  - Consider removing host from alert scope to reduce separate noise.
  - No Warning threshold. 
- Alert Events
  - `source:alert @monitor_notifications:*xxinfra-alerts* status:error @monitor_id:60714933`
  - Top Multi-Host Alert Events: past 1mo, count by instance, top 10, cutoff_min 4, rollup every 1h

### Container Events

- `service:containerd status:error "Failed to handle backOff event" env:prod`
  - Pie chart by Count of Instance with sorted legend by percent descending
