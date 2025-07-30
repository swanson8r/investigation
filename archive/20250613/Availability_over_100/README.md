# Investigation: Availability > 100%

## Notebook

### markdown widget

`#ask-dev-sre` slack thread investigating missing data in the range of

- `Mon Sept 16th 7am ET -> Tues Sept 17th 4am ET`
  - `(Sep 16, 06:00 am – Sep 17, 5:00 am)`

has "ears" of nonimpacting time around the event for comparison against "normal"

### Timeseries widgets - Line Charts

#### Timeseries Component Metrics: Success and Valid: Other Path Availability - "1d" 1d Sep 16 - Sep 17

- Not the problem; Shows 1:1 mapping

#### MISSING DATA Timeseries Component Metrics: Success and Valid: Other Path Availability - "1d"

- Problem: long time range queries (Monthly Availability) not returning all `http.status_class` values due to sampling
- Solutions:
  - filter only on service, not on http metrics
  - If status_class must be used, use IN instead of NOT IN for explicit selection and reduced cardinality of returned values.
  - NEW Correlated Log Subquery to address this exact issue:
    - https://www.datadoghq.com/blog/filter-logs-by-subqueries-with-datadog 
    - https://docs.datadoghq.com/logs/explorer/advanced_search/#filter-logs-with-subqueries
  - NEW Nested Queries for [Higher resolution queries over historical time frames](https://docs.datadoghq.com/metrics/nested_queries/#higher-resolution-queries-over-historical-time-frames)
    - Datadog recommends that you define your initial rollup with the most granular rollup interval and use multilayer time aggregation with coarser rollup intervals to get more user-readable graphs

#### MISSING DATA WORKAROUND Success and Valid: Other Path Availability - "1d"

- No more mismatched metrics
- Partial query: `sum:trace.nginx.request.hits.by_http_status{service:api-gateway AND http.status_class NOT IN (4xx,5xx) AND resource_name:*_/api/v*/<path>`
