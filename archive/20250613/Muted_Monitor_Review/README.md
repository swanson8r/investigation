# Muted Monitor Review

## Notebook

### Markdown Widget

#### $team Monitors

- Production - kube-proxy Replica Availability
  - env is inconsistently populated in KES metrics. pending updated tags or decommission.
  - customer env is right on the 80% threshold
- Production - High Disk Utilization
  - no env group by
  - mostly dev and pr instances impacted
  - WAY too many groups returned
  - simplify by env and instance, or env and host?
- 60 pages of downtimes (by host) on this one monitor x ~ 5 per page
- `Datadog Consolidation - Pointing devspace & PR clusters to the $tenant Datadog account.
  - Monitor tags: *
  - Scope: `instance:dev OR instance:pr OR instance:azure-dev
  - Confirm with $team if this should be baked into the monitor; break out a separate one for dev & pr if needed.

#### Evaluate 15w downtime for removal

- NO DATA or Alerting, sorted by Muted Elapsed descending

Sunday, 3/23/25

- Focused on a refined filter on muted monitors since 15 weeks ago, managed by terraform
- Initial Query only show muted for *at least* this long. We want the opposite.
- Show a list of all tags being filtered by the downtime exclusion.
  - are any of them valid?
  - Invalid query when pasted directly into triggers
  - `group:"instance:*" AND NOT group:"instance:prod*"` returns nothing
  - `group:instance:prod*` (no quotes) returns 55 correct looking triggered anti-patterns.

this works to start with (614 results) `-group"dbt_instance:prod*`

#### Monitor List Search

To further narrow the search, start with the raw list of monitors by tags.

- `muted:true tag:("managed-by:terraform") -tag:("initiative:observability") tag:(domain:*) -ExternalSecrets`
  - returns 96 legacy domain monitors to exclude (recently downtimed by group scope)
- `tag:"managed-by:terraform" -tag:("initiative:observability") -tag:(domain:*) -ExternalSecrets notification:webhook-incident-io -muted_elapsed:16w muted_elapsed:14w`
  - gets us down to 82 monitors muted by this downtime to explore for overly broad scope.
  - Filtering further on exclude status OK, only 1 is alerting
