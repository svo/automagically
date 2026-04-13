---
name: workflow-schema
description: Manages the Automagically workflow JSON schema — the shared contract between the NL parser, workflow definition service, UI diagram renderer, and execution engine. Use when viewing, modifying, or validating the workflow schema, or when understanding how workflows are represented as data.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Workflow Schema

The workflow JSON schema is the most critical shared contract in the platform. It defines how workflows are represented as data and is consumed by four components:

- **nl-parser** produces it from natural language
- **workflow-definition** stores and versions it
- **ui** renders it as a visual diagram
- **execution-engine** executes it

## Schema location

The canonical schema lives at `repository/workflow-definition/` and is referenced by the other services.

## Schema structure

A workflow is a DAG (directed acyclic graph):

```json
{
  "id": "uuid",
  "name": "string",
  "version": "integer",
  "nodes": [
    {
      "id": "uuid",
      "type": "trigger | condition | action",
      "connector": "string (connector name from catalog)",
      "operation": "string (trigger or action name)",
      "config": {
        "...connector-specific configuration"
      },
      "position": { "x": "number", "y": "number" }
    }
  ],
  "edges": [
    {
      "id": "uuid",
      "source": "node-id",
      "target": "node-id",
      "condition": "optional expression for conditional edges"
    }
  ],
  "metadata": {
    "created_at": "iso8601",
    "updated_at": "iso8601",
    "created_by": "user-id"
  }
}
```

## Modification protocol

Changes to this schema are high-impact. When modifying:

1. Update the canonical schema in workflow-definition
2. Update the NL parser's output format
3. Update the UI's type definitions and renderer
4. Update the execution engine's input parser
5. Add migration logic for existing stored workflows
6. Update tests in all four services

Always discuss schema changes before implementing them. Use `/specify` to create a spec first.

## Validation

The workflow-definition service validates workflows on save. Key rules:
- All edge source/target IDs must reference existing nodes
- Exactly one trigger node per workflow
- No cycles in the graph
- All connector references must exist in the connector-catalog
- All required config fields per the connector schema must be present
