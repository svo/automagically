# Automagically

A web-based automation platform that allows users to build automation workflows visually and via natural language input. Describe an automation in plain English and the platform generates a visual workflow diagram of triggers, conditions, and actions. Users can also build and edit workflows directly on a visual canvas.

Comparable to n8n, Zapier, Make, and Atlassian Automation.

## Architecture

All services and the frontend follow **hexagonal (ports and adapters) architecture**. Services run locally via Vagrant with the Docker provider.

- **Python microservices** are based on `reference/python-sprint-zero`
- **Frontend** is based on `reference/www-qual-is` (Next.js 15 / React 19 / TypeScript / Tailwind)

### Services

| Service | Container Port | Host Port | Description |
|---|---|---|---|
| workflow-definition | 8001 | 28001 | CRUD and versioning for workflow DAGs |
| execution-engine | 8002 | 28002 | DAG resolution, node execution, retry and error handling |
| trigger | 8003 | 28003 | Cron, webhook, polling, and event-based triggers |
| connector-catalog | 8004 | 28004 | Registry of integrations, triggers, actions, and config schemas |
| connector-runtime | 8005 | 28005 | Plugin host that executes integration actions |
| credential-store | 8006 | 28006 | Encrypted secret management and OAuth flows |
| execution-log | 8007 | 28007 | Run history, per-node input/output, timing, status |
| user-workspace | 8008 | 28008 | Authentication, teams, workspace scoping, permissions |
| nl-parser | 8009 | 28009 | Natural language to structured workflow definition via LLM |
| api-gateway | 8010 | 28010 | Routing and auth middleware |
| ui | 3000 | 23000 | Visual workflow canvas, NL input, execution history |

## Repository Structure

```
automagically/
├── .claude/
│   ├── CLAUDE.md
│   └── skills/
├── Vagrantfile
├── reference/
│   ├── python-sprint-zero/     # Python service template
│   └── www-qual-is/            # Frontend template
└── repository/
    ├── workflow-definition/
    ├── execution-engine/
    ├── trigger/
    ├── connector-catalog/
    ├── connector-runtime/
    ├── credential-store/
    ├── execution-log/
    ├── user-workspace/
    ├── nl-parser/
    ├── api-gateway/
    └── ui/
```

All services and the frontend are git submodules. The `reference/` directory contains the template repositories used to scaffold new services.

## Getting Started

```bash
git clone --recurse-submodules <repo-url>
cd automagically
vagrant up
```

Or if already cloned:

```bash
git submodule update --init --recursive
```

## Claude Code Skills

This project includes Claude Code skills in `.claude/skills/` that assist with development tasks. Skills are either **automatically triggered** by Claude when the context matches, or **manually invoked** via slash commands.

### Automatic Skills

These skills are triggered by Claude when it detects a relevant context — no slash command needed.

| Skill | Description |
|---|---|
| **explore-architecture** | Reads CLAUDE.md files across all submodules and summarises the current state of each service. Triggered during onboarding, dependency analysis, or codebase audits. |
| **workflow-schema** | Manages the workflow JSON schema — the shared contract between the NL parser, workflow definition service, UI renderer, and execution engine. Triggered when viewing, modifying, or validating the schema. |
| **sequential-thinking** | Breaks down complex problems into numbered reasoning steps with revision, branching, hypothesis generation, and verification. Triggered for multi-step analysis or architectural decisions. |

### Manual Skills

These skills are invoked explicitly via slash commands.

| Skill | Command | Description |
|---|---|---|
| **scaffold-services** | `/scaffold-services <name> <template> <port> <description>` | Creates a new service repo from a template and adds it as a submodule. Handles repo creation, template renaming, port assignment, and Vagrantfile updates. |
| **specify** | `/specify <feature-description>` | Defines a feature specification with user stories, acceptance criteria, and domain model impacts. Produces a `SPEC.md` under `.specs/`. |
| **plan** | `/plan <spec-name>` | Creates a technical implementation plan from a feature specification. Produces a `PLAN.md` with per-service tasks and cross-service coordination points. |
| **add-connector** | `/add-connector <connector-name>` | Scaffolds a new connector integration across the connector-catalog and connector-runtime services. |
| **run-service** | `/run-service <service-name\|all>` | Starts one or more services locally using their Docker images. Resolves image names from each service's Packer config. |
| **health-check** | `/health-check` | Checks the health and status of all services. Reports which are running, reachable, and responding on their expected ports. |
