## Notebook

**Datadog Agent Startup Investigation**

<NEW> Spike in alerts may be handled with env vars
- `DD_CLUSTER_CHECKS_WARMUP_DURATION` https://docs.datadoghq.com/containers/cluster_agent/commands/?tab=helm
- Two others that I found a while back and now escape me

### Timeseries

- **Count by env over "source:datadog @evt.type:"Agent Startup""
  - `env in cutoff_max(exclude_null(count[source:datadog @evt.type:"Agent Startup"]), 30)`

### Log Patterns

** Agent Errors **

- Datadog Agent Error Log Patterns, past 3 days
  - Count: 136K, Volume: 15K. @CHECK:redisdb
  - Message: Error running check: [{"message":"Authentication required.","traceback":"

