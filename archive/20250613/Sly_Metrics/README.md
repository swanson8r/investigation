# Observability Service Level Objectives Metrics

## Notebook

- The long way of saying SLIs

### Pie Chart Widgets

#### Distribution of service: tag values in $tenant_type logs by count, past 7 days

- Cloned to show total (log file? index?) byte size by service
- Cloned again to show by env: tag value

#### RabbitMQ Consumer Count by Queue Name

### Timeseries Widgets

#### Unparsed Logs by Service past 2 weeks

- `service in sum:logs.unparsed{*}.as_count()`
  - custom metric: https://docs.datadoghq.com/logs/guide/detect-unparsed-logs/#create-a-metric-to-track-for-unparsed-logs
  - Make sure "Show N/A" is selected. Might need to do by `source` if service is not tagged / autodetected.
- line chart; can infer team based on context clues and update container agent log processing config
  - Investigate why this was not included in autodiscovery

#### $service rate of Ingested Events by status, past 2 weeks

- status in Log Events Ingested by Service
  - high instance of info logs should prompt query of `level` or `message` contents with errors / tracebacks, and pattern review for exclusion filter
