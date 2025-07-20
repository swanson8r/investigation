# Investigation

## Problem Statement

In the Datadog Learning Center: [Datadog API: Automation and Infrastructure as Code](https://learn.datadoghq.com/courses/dd-api-automation-iac) > [Lab: Command Line](https://learn.datadoghq.com/courses/take/dd-api-automation-iac/texts/35844608-lab-command-line),
docker container `redis-session-cache` is running by default, but the lab instructions expect it to require a manual start.

Symptoms shared in [Datadog Community Slack](https://chat.datadoghq.com/), [#learning-center](https://datadoghq.slack.com/archives/CAN0MS5K6/p1752946866940809) channel.

## Validation

1. Confirm redis service is up and for how long
   ```bash
   Checking Terminal 2:
   root@lab-host:~/lab# docker ps -a --filter "name=redis-session-cache"
   CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS      NAMES
   b68e1a98728d   redis:7.4.2-alpine   "docker-entrypoint.s…"   34 minutes ago   Up 34 minutes   6379/tcp   redis-session-cache
   ```

## Artifacts

### docker-compose.yml 

Available in the IDE, shows 2 services configured. The other container name is `stately`.

## Hypothesis

1. It is possible an instruqt startup script is executing `docker compose up` and redis-session-cache is coming along for the ride

## Troubleshooting

1. Configure Autodiscovery host logging to capture startup scripts on the linux host. Correlate with existing Container and / or Redis integration for timing
2. Once the source of autostart has been discovered, exclude the redis container.
