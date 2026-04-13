# Automagically Platform

## Project Purpose

Automagically is a web-based automation platform (similar to n8n, Zapier, Make, and Atlassian Automation) that allows users to build automation workflows visually and via natural language input. A user describes an automation in plain English and the platform generates a visual workflow diagram of triggers, conditions, and actions. Users can also build and edit workflows directly on a visual canvas.

This is the **parent repository** containing all services and the frontend as git submodules. It is a local-only spike — all services run on localhost via Vagrant with the Docker provider.

### Core Domain Concepts

- **Workflow**: A directed acyclic graph (DAG) of nodes and edges representing an automation
- **Node**: A single step in a workflow — a trigger, condition, or action
- **Edge**: A connection between nodes defining data flow and execution order
- **Trigger**: The event that starts a workflow (cron, webhook, polling, or event-based)
- **Action**: An operation performed by a connector (send email, call API, etc.)
- **Connector**: An integration with an external service, defining available triggers and actions
- **Credential**: An encrypted secret used to authenticate with external services
- **Execution**: A single run of a workflow, recording per-node input/output and status

## Architecture

### Hexagonal Architecture

All services and the frontend follow hexagonal (ports and adapters) architecture. Domain logic is pure and has no external dependencies. Infrastructure concerns (databases, HTTP clients, message queues) are adapters behind port interfaces.

- **Python microservices** are based on `reference/python-sprint-zero`
- **Frontend** is based on `reference/www-qual-is` (Next.js 15 / React 19 / TypeScript / Tailwind)

### Repository Structure

```
automagically-platform/
├── .claude/
│   ├── CLAUDE.md
│   └── skills/
├── Vagrantfile
├── reference/
│   ├── python-sprint-zero/     # Python service template
│   └── www-qual-is/            # Frontend template
└── repository/
    ├── workflow-definition/    # Port 8001
    ├── execution-engine/       # Port 8002
    ├── trigger/                # Port 8003
    ├── connector-catalog/      # Port 8004
    ├── connector-runtime/      # Port 8005
    ├── credential-store/       # Port 8006
    ├── execution-log/          # Port 8007
    ├── user-workspace/         # Port 8008
    ├── nl-parser/              # Port 8009
    ├── api-gateway/            # Port 8010
    └── ui/                     # Port 3000
```

### Port Assignments

Each service runs in its own Docker container managed by Vagrant. The Vagrantfile maps container ports to the host with a `2` prefix:

| Service              | Container Port | Host Port |
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
| ui (Next.js)         | 3000    | 23000     |

The frontend calls backend services on their host-mapped ports (e.g. `http://localhost:28001`).

### Service Responsibilities

- **workflow-definition**: CRUD and versioning for workflow DAGs (nodes, edges, metadata)
- **execution-engine**: DAG resolution, sequential node execution, data passing, retry and error handling
- **trigger**: Cron/schedule, webhook, polling, and event-based workflow triggers
- **connector-catalog**: Registry of available integrations, their triggers, actions, and config schemas
- **connector-runtime**: Plugin host that executes integration actions (email, HTTP, Slack, AWS, etc.)
- **credential-store**: Encrypted secret management, OAuth flows, credential testing
- **execution-log**: Run history, per-node input/output, timing, status
- **user-workspace**: Authentication, teams, workspace scoping, permissions
- **nl-parser**: Natural language to structured workflow definition via LLM, grounded against the connector catalog
- **api-gateway**: Routing and auth middleware (lightweight for local dev)
- **ui**: Visual workflow canvas, NL input, execution history, credential management

### Docker Images

Each service produces a Docker image. The image name is defined by the `docker-tag` variable in the service's `infrastructure/packer/service.pkr.hcl`. To find a service's image name:

```bash
grep docker-tag repository/<service-name>/infrastructure/packer/service.pkr.hcl
```

Services are run locally via Vagrant with the Docker provider. The Vagrantfile port mappings forward container ports to the host.

## Development Conventions

### Working with Submodules

All services and the frontend are git submodules. After cloning:

```bash
git submodule update --init --recursive
```

Each submodule has its own `.claude/CLAUDE.md` with service-specific context. When working on a specific service, read its CLAUDE.md first.

### Cross-Service Changes

When making changes that span multiple services (e.g. modifying the workflow schema), commit to each submodule individually, then update the submodule references in the parent repo.

### Workflow JSON Schema

The workflow definition JSON schema is the contract between the NL parser (which produces it), the workflow definition service (which stores it), the diagram renderer in the UI (which displays it), and the execution engine (which runs it). Changes to this schema require coordinated updates across all four.

### Testing

Each submodule follows the testing conventions of its template:
- Python services: pytest with the structure from python-sprint-zero
- Frontend: Vitest (unit) and Playwright (E2E) from www-qual-is

### Code Style

- Python services follow the linting and formatting rules from python-sprint-zero
- Frontend follows ESLint and Prettier rules from www-qual-is
- Architectural boundary enforcement via dependency-cruiser (frontend) and equivalent rules in Python services
