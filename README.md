# runpodctl-project

This repo's purpose to provide a simple reproducable testcase for `runpodctl project deploy`

## Changes in the Dockerfile

In order to develop a custom worker using `runpodctl project dev`, I needed to overwrite Dockerfile using `runpod/base:0.6.1-cuda12.2.0"` as the base. When using `CMD /start.sh` everything works as expected. When using `ENTRYPOINT ["/start.sh"]` the worker also works during development, but fails during deployment as a serverless endpoint. It seems that the ENTRYPOINT overwrites the `container start command` in the template that is generated after running `runpodctl project deploy`.

## Steps to reproduce

1. Clone this repository
2. Run `runpodctl project deploy`
3. Create a request to the deployed endpoint:

```
curl -X POST https://api.runpod.ai/v2/ENDPOINT_ID/runsync \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer RUNPOD_API_KEY" \
  -d '{"input": {"name": "world"}}'
```

4. See that the request keeps for every in state `IN_QUEUE`
5. See in the container logs, that the worker started nginx and fails to start ssh. Both of which should not be started, as the container start command in the template should overwrite this.

```
2024-10-10T14:22:46.247536458Z Starting Nginx service...
2024-10-10T14:22:46.268940119Z  * Starting nginx nginx
2024-10-10T14:22:46.277866256Z    ...done.
2024-10-10T14:22:46.278477804Z Pod Started
2024-10-10T14:22:46.278555257Z Exporting environment variables...
2024-10-10T14:22:46.283013304Z Running post-start script...
2024-10-10T14:22:46.286223554Z Monitoring SSH connections every 60 seconds, with a countdown of 120 seconds.
2024-10-10T14:22:46.286529733Z Start script(s) finished, pod is ready to use.
2024-10-10T14:22:46.294703246Z 2024/10/10 14:22:46 No config file used
2024-10-10T14:22:46.367562078Z 2024/10/10 14:22:46 Listening on [::]:4040
2024-10-10T14:23:46.343980154Z No SSH connections found. Countdown: 60 seconds remaining.
```

## Create the Docker image

```
docker build --platform linux/amd64 -t timpietruskyrunpod/runpodctl-project-deploy-entrypoint-error/latest .
docker push timpietruskyrunpod/runpodctl-project-deploy-entrypoint-error/:latest
```
