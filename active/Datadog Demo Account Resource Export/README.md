# Investigation: Datadog Demo Account Resource Export

## Problem Statement

The Datadog Learning Labs create a demo account that expires in 14 days.
With one day left in the current demo account, a recent lab created a new one instead of continuing to use the existing one.
It would be of value to export all resources created in Datadog before access to this account lapses

## Observations

1. Infrastructure as Code options include the official Terraform Datadog Provider, which supports _some_ resource types.
2. NoBS has created datadog-terraform-quick-start to automate the export of resources

## Analysis

1. Identify resources actively using metrics
   - [Metrics Summary Filtered Search](https://app.datadoghq.com/metric/summary?facet.query_activity=-not_queried_thirty_days%2C-not_queried_sixty_days%2C-not_queried_ninety_days&facet.related_assets=-no_related_assets&window=1209600)
2. Fork datadog-terraform-quick-start and Clone locally
3. Attempt to run script with demo account credentials
   - Instructions at https://github.com/swanson8r/datadog-terraform-quick-start
   - First pass: users and resources only
         ```yaml
         resources:
           - role: []
           - user: []
         ````
4. Move or delete terraform folder contents before re-running export.
   - `rm -rf terraform/datadog/*`
5. Identify any errors and proceed to Troubleshooting to iterate.
   - Second Pass: all resources
         ```yaml
         resources:
           - all:
         ```
   - Error attempting to export all resources
           ```python
           2025-07-30 19:05:10,189 - INFO - Found "all" in configuration; importing all supported resources.
           Traceback (most recent call last):
           File "//code/configure.py", line 210, in <module>
           write_command("--resources=*")
           TypeError: write_command() missing 1 required positional argument: 'resource'
           ```
     - Identified code failure when applying `write_command("--resources=*")`
       - `def write_command` had no default value for the first argument (path).
         - Arguments are passed in unnamed, so are positional. If only one argument is provided, it will be assumed as a path with no resources requested.
     - next iteration of error
           ```python
           2025-07-30 19:25:25,447 - INFO - Found "all" in configuration; importing all supported resources.
            //code/configure.py:164: DeprecationWarning: The 'warn' method is deprecated, use 'warning' instead
            logger.warn(
            2025-07-30 19:25:25,457 - WARNING - Command "/usr/local/bin/terraformer import datadog -n 5 -m 1000 -p  --resources=--resources="*"" failed with error 2025/07/30 19:25:25 required flag(s) "resources" not set
            ```

## Troubleshooting

1. `docker compose build` for macOS assumes Docker Desktop is already running
2. yaml config required some formatting (`[]`) to address empty / null values
3. More explicit steps needed for clearing out previous terraform output in datadog folder before rerunning
4. Pass an empty path when calling `write_command` for all resources.
5. Omit the already defined `--resources` flag when requesting all with `"*"`
6. Provide a default path when requesting all resources. `write_command("{provider}/{service}","*")`
7. Update to `logger.warning` to remove deprecation messages

## Solution Statement

Created PR against branch of fork after successfully executing export of all resources.
