---
name: run-service
description: Starts one or more Automagically services locally using their Docker images. Reads the docker-tag from each service's infrastructure/packer/service.pkr.hcl to determine the image name. Use when the user wants to run, start, or restart a specific service.
disable-model-invocation: true
allowed-tools: Bash(*)
---

# Run Service

Start an Automagically service locally using its Docker image.

## Usage

`/run-service <service-name>` or `/run-service all`

## Port Assignments

| Service              | VM Port | Host Port |
|----------------------|---------|-----------|
| workflow-definition  | 8001    | 28001     |
| execution-engine     | 8002    | 28002     |
| trigger              | 8003    | 28003     |
| connector-catalog    | 8004    | 28004     |
| connector-runtime    | 8005    | 28005     |
| credential-store     | 8006    | 28006     |
| execution-log        | 8007    | 28007     |
| user-workspace       | 8008    | 28008     |
| nl-parser            | 8009    | 28009     |
| api-gateway          | 8010    | 28010     |
| ui                   | 3000    | 23000     |

## Resolving the Docker image name

Each service's image name is defined by the `docker-tag` variable in its Packer config:

```bash
docker_tag=$(grep -oP 'docker-tag\s*=\s*"\K[^"]+' repository/$0/infrastructure/packer/service.pkr.hcl)
```

## Steps for Python services

1. Resolve the Docker image name from `repository/$0/infrastructure/packer/service.pkr.hcl`.

2. Build the image if not already built:
   ```bash
   cd repository/$0
   docker build -t ${docker_tag} .
   ```

3. Run the container on the assigned port:
   ```bash
   docker run -d \
     --name automagically-$0 \
     -p $PORT:$PORT \
     -e PORT=$PORT \
     -e HOST=0.0.0.0 \
     ${docker_tag}
   ```

   Look up `$PORT` from the table above.

## Steps for the frontend

1. Resolve the Docker image name from `repository/ui/infrastructure/packer/service.pkr.hcl`.

2. Build and run:
   ```bash
   cd repository/ui
   docker build -t ${docker_tag} .
   docker run -d \
     --name automagically-ui \
     -p 3000:3000 \
     -e PORT=3000 \
     -e NEXT_PUBLIC_WORKFLOW_DEFINITION_URL=http://localhost:28001 \
     -e NEXT_PUBLIC_EXECUTION_ENGINE_URL=http://localhost:28002 \
     -e NEXT_PUBLIC_TRIGGER_URL=http://localhost:28003 \
     -e NEXT_PUBLIC_CONNECTOR_CATALOG_URL=http://localhost:28004 \
     -e NEXT_PUBLIC_CONNECTOR_RUNTIME_URL=http://localhost:28005 \
     -e NEXT_PUBLIC_CREDENTIAL_STORE_URL=http://localhost:28006 \
     -e NEXT_PUBLIC_EXECUTION_LOG_URL=http://localhost:28007 \
     -e NEXT_PUBLIC_USER_WORKSPACE_URL=http://localhost:28008 \
     -e NEXT_PUBLIC_NL_PARSER_URL=http://localhost:28009 \
     -e NEXT_PUBLIC_API_GATEWAY_URL=http://localhost:28010 \
     ${docker_tag}
   ```

## Running all services

If `$0` is `all`, iterate over each service directory, resolve its `docker-tag`, build and run each container in sequence. Start Python services first, then the frontend. Report which containers are running and on which ports.

## Stopping services

```bash
docker stop automagically-$0 && docker rm automagically-$0
```

Or to stop all:
```bash
docker ps --filter "name=automagically-" -q | xargs docker stop | xargs docker rm
```
