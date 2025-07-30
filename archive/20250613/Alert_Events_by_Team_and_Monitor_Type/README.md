# Alert Events by Team and Monitor Type

## Dashboard

### Pie Charts

- Count `by team, @monitor_notifications` over
      ```datadog
       "@monitor.tags:"managed-by:terraform" @monitor_notifications:*
      ```
- A few top teams with the rest in the noise
  - channels per team as their own slice of the pie
- filter on `@evt.type` to narrow to specific monitor types

### Top List

- Count `by @evt.type, team` over
      ```datadog
       "source:alert @monitor.tags:"managed-by:terraform" team:* status:error service:* -@monitor.tags<...>
      ```
- `query_alert_monitor` at top, legend of teams below
- Count `by team, @monitor_notifications` over
      ```datadog
      "@monitor.tags:"managed-by:terraform" @monitor_notifications:*
      ```
- Simple Top List of Team and notification channel w/o legend
