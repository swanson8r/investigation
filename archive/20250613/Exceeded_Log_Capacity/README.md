# monitor 168690753 Exceeded Datadog Log Capacity

## Notebook

### Markdown widget

#### Production - Exceeded Datadog Log Capacity

- `events("source:datadog datadog_index:main quota").rollup("count").last("5m") >= 1`
- Has Never Alerted (no history in slack)
- OK Monitor status since 98 weeks

### Timeseries Bar Charts

#### Monitor Metrics: Approaching Datadog Max Log Capacity - by Status

- `status in sum:datadog.estimated_usage.logs.ingested_events{datadog_is_excluded:false,datadog_index:main,child_org_name:self`

#### Estimated Usage - Ingested Events by Datadog Index Name. Daily Sum

#### Estimated Usage - Ingested Events for Status: Emergency. Daily Sum by Service, Index name (all are Main)

- Shows unconfigured log status mapping
  - Discovered missing JSON Preprocessor mapping for status = level in the consolidated Org
