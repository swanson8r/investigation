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

   root@lab-host:~/lab# cat /etc/os* | head -4
   PRETTY_NAME="Ubuntu 22.04.5 LTS"
   NAME="Ubuntu"
   VERSION_ID="22.04"
   VERSION="22.04.5 LTS (Jammy Jellyfish)"
   ```

## Artifacts

### termainal commands

- Install linux host agent
  ```bash
  DD_API_KEY=${DD_API_KEY} \
  DD_SITE="datadoghq.com" \
  DD_REMOTE_UPDATES=true \
  DD_ENV=api-course \
  bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
  ```
- Enable docker Agent APM
  ```bash
  DD_APM_INSTRUMENTATION_LIBRARIES=java:1,python:3,js:5,php:1,dotnet:3 DD_APM_INSTRUMENTATION_ENABLED=docker DD_NO_AGENT_INSTALL=true bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
  ```
- Remove old Docker agent
  ```bash
  docker stop dd-agent; docker rm dd-agent
  ```
- Install Docker agent
  ```bash
  docker run -d --name dd-agent \
   -e DD_API_KEY=${DD_API_KEY} \
   -e DD_SITE="datadoghq.com" \
   -e DD_DOGSTATSD_NON_LOCAL_TRAFFIC=true \
   -e DD_ENV=api-course \
   -e DD_APM_ENABLED=true \
   -e DD_APM_NON_LOCAL_TRAFFIC=true \
   -e DD_APM_RECEIVER_SOCKET=/var/run/datadog/apm.socket \
   -e DD_DOGSTATSD_SOCKET=/var/run/datadog/dsd.socket \
   -v /var/run/datadog:/var/run/datadog \
   -e DD_LOGS_ENABLED=true \
   -e DD_LOGS_CONFIG_CONTAINER_COLLECT_ALL=true \
   -e DD_CONTAINER_EXCLUDE="name:datadog-agent" \
   -v /opt/datadog-agent/run:/opt/datadog-agent/run:rw \
   -v /var/run/docker.sock:/var/run/docker.sock:ro \
   -v /proc/:/host/proc/:ro \
   -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
   -v /var/lib/docker/containers:/var/lib/docker/containers:ro \
   gcr.io/datadoghq/agent:7
  ```

  - poke around in redis container shell
    `docker exec -it redis-session-cache sh`
  
### docker-compose.yml 

Available in the IDE, shows 2 services configured. The other container name is `stately`.

### Datadog UI

#### Infrastructure

- [Live Containers](https://app.datadoghq.com/containers?saved-view-id=3663776) Saved View in Lab account, past 15 minutes
  - redis-session-cache **UP**
  - [redis Container > Layers](https://app.datadoghq.com/container-images?query=&inspect=redis%40sha256%3A02419de7eddf55aa5bcf49efb74e88fa8d931b4d77c07eff8a6b2144472b6952&multiArchFilter=amd64%2Flinux&panelTab=layers) ([Preview Feature](https://www.datadoghq.com/blog/missing-container-metadata/)
    - `10. RUN /bin/sh -c mkdir /data && chown redis:redis /data # buildkit`
    - `13. COPY docker-entrypoint.sh /usr/local/bin/ # buildkit`
    - `16. CMD ["redis-server"]`

#### Logs

- [Container Logs](https://app.datadoghq.com/logs?saved-view-id=3663810) `host:api-course-host container_name:redis-session-cache`
  - [Warning](https://app.datadoghq.com/logs?query=host%3Aapi-course-host%20container_name%3Aredis-session-cache&agg_m=count&agg_m_source=base&agg_q=source&agg_q_source=base&agg_t=count&cols=host%2Cservice&event=AwAAAZgqPOjG-83npQAAABhBWmdxUFJkZUFBQ0ZnWlJvRTJnRThRRXIAAAAkZjE5ODJhNDItNjgxNy00MWRjLTgyMTMtNTQzMzVhMTY2YTFkAAAB8A&fromUser=true&messageDisplay=inline&refresh_mode=sliding&saved-view-id=3663810&storage=hot&stream_sort=time%2Cdesc&top_n=10&top_o=top&viz=stream&x_missing=true&from_ts=1753054721244&to_ts=1753055621244&live=true): no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
  - [Notice](https://app.datadoghq.com/logs?query=container_id%3Aae53c4066f561cb95ad1f2d2ba7f5cc5bef305a8d288fed139023ea1c672260e&agg_m=count&agg_m_source=base&agg_t=count&cols=host%2Cservice&event=AwAAAZgqPOjG-83npAAAABhBWmdxUFJkZUFBQ0ZnWlJvRTJnRThRRXEAAAAkZjE5ODJhNDItNjgxNy00MWRjLTgyMTMtNTQzMzVhMTY2YTFkAAAB7Q&fromUser=true&messageDisplay=inline&refresh_mode=sliding&storage=hot&stream_sort=desc&viz=stream&from_ts=1753054582953&to_ts=1753055482953&live=true) Redis version=7.4.2, bits=64, commit=00000000, modified=0, pid=1, just started
- [Host Logs](https://app.datadoghq.com/logs?saved-view-id=3663814)  `host:api-course-host docker`
  - `2025-07-20 23:48:13 UTC | PROCESS | INFO | (pkg/util/log/log.go:845 in func1) | 1 Features detected from environment: docker`
    - `process: func1`
 - `2025-07-21 00:59:56 UTC | CORE | INFO | (pkg/logs/launchers/file/launcher.go:325 in startNewTailer) | Starting a new tailer for: /var/lib/docker/containers/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc-json.log (offset: 0, whence: 0) for tailer key /var/lib/docker/containers/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc-json.log/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc`
 - `2025-07-21 00:59:56 UTC | CORE | INFO | (pkg/logs/tailers/file/tailer_nix.go:30 in setup) | Opening /var/lib/docker/containers/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc-json.log for tailer key /var/lib/docker/containers/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc-json.log/41085964b343d21d590f35810b3b12d061c3036c0187e40f41e09b491f60bbcc`
   - `dirname:/var/lib/docker/containers/9aecc97b48a7ca3a8e7380af8850e604a7045b24fb6d323993c3da0440d8ed54`
   - `filename:9aecc97b48a7ca3a8e7380af8850e604a7045b24fb6d323993c3da0440d8ed54-json.log`

#### Integrations

##### Fleet Automation

- [Agent Info](https://app.datadoghq.com/fleet?query=api-course-host&sp=%5B%7B%22p%22%3A%7B%22agentKey%22%3A%22f4217785fd2dbfcd067dab235f1e3713%22%2C%22tab%22%3A%22info%22%7D%2C%22i%22%3A%22fleet_agent-details%22%7D%5D)
  - `api-course-host`
  - Linux Agent Version: 7.64.2
    - Remote Agent Management: Unsupported
    - (Use the generated Agent installation command to upgrade your Agent to version 7.66+.)
 - `lab-host.ida5ib5myjmo.svc.cluster.local`
   - Docker agent
    
##### Redis

- log error messages
  ```bash
  2025-07-21 01:41:00 UTC | CORE | ERROR | (pkg/collector/worker/check_logger.go:71 in Error) | check:redisdb | Error running check: [{"message":"Timeout connecting to server","traceback":"Traceback (most recent call last): 
  ```


##### Audit Events

- Agent version was updated to 7.64.2 from 7.68.1

## Hypothesis

1. It is possible an instruqt startup script is executing `docker compose up` and redis-session-cache is coming along for the ride

## Troubleshooting

1. Enable Audit Trail for agent change monitoring
2. [Install / Upgrade Linux Host Agent](https://app.datadoghq.com/fleet/install-agent/latest?platform=linux) with [Enable Remote Management](https://docs.datadoghq.com/agent/fleet_automation/remote_management/#setup) set to true
   - Note: This lab will timeout after 10 minutes of inactivity.
   - [Create a new Upgrade Deployment](https://app.datadoghq.com/fleet/create-upgrade-deployment?from=%2Ffleet%2Fagent-upgrades)
3. [Install / Upgrade Docker Agent](https://docs.datadoghq.com/containers/docker/?tab=standard)
   - Note: Run only one Datadog Agent per node to avoid unexpected behavior.
   - Remotely upgrading Agents in containerized environments is not supported.
4. Check Redis Integration permission groups for docker agent
5. Make sure to look in the right place; `lab-host` = `api-course-host` running agent inside the dd-agent container on Ubuntu 22.04 host. Alpine Linux logs are from inside the redis docker container under `/var/lib/docker/containers/`.
6. Configure Autodiscovery host logging to capture startup scripts on the linux host. Correlate with existing Container and / or Redis integration for timing
7. Once the source of autostart has been discovered, exclude the redis container.
