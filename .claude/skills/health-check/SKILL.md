---
name: health-check
description: Checks the health and status of all Automagically services. Reports which services are running, reachable, and responding on their expected ports. Use when debugging connectivity issues or verifying the platform is up.
disable-model-invocation: true
allowed-tools: Bash(curl *), Bash(lsof *), Bash(ps *), Bash(docker *), Bash(grep *)
---

# Health Check

Verify that all Automagically services are running and reachable.

## Usage

`/health-check`

## Process

1. Check Docker container status for all Automagically services:

```bash
docker ps --filter "name=automagically-" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

2. For each service, check if the port is reachable:

```bash
for port in 8001 8002 8003 8004 8005 8006 8007 8008 8009 8010 3000; do
  curl -s -o /dev/null -w "%{http_code}" http://localhost:${port}/health || echo "DOWN"
done
```

3. Report the status of each service in a table:

| Service              | Port  | Status |
|----------------------|-------|--------|
| workflow-definition  | 8001  | ✓ / ✗  |
| execution-engine     | 8002  | ✓ / ✗  |
| trigger              | 8003  | ✓ / ✗  |
| connector-catalog    | 8004  | ✓ / ✗  |
| connector-runtime    | 8005  | ✓ / ✗  |
| credential-store     | 8006  | ✓ / ✗  |
| execution-log        | 8007  | ✓ / ✗  |
| user-workspace       | 8008  | ✓ / ✗  |
| nl-parser            | 8009  | ✓ / ✗  |
| api-gateway          | 8010  | ✓ / ✗  |
| ui                   | 3000  | ✓ / ✗  |

4. If running from the host (outside the VM), use the host-mapped ports (28001–28010, 23000) instead.

5. For any services that are down, check if the Docker container exists but is stopped (`docker ps -a --filter "name=automagically-$service"`), and suggest `/run-service` to start them.

6. To verify the correct image is running, resolve the expected `docker-tag` from each service's `infrastructure/packer/service.pkr.hcl` and compare against the running container's image.
