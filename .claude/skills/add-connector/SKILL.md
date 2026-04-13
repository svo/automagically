---
name: add-connector
description: Adds a new connector integration to the Automagically platform. Scaffolds the connector definition in the connector-catalog service and the adapter implementation in the connector-runtime service. Use when adding support for a new external service like Slack, GitHub, AWS, SMTP, etc.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash(*), Glob, Grep
---

# Add Connector

Scaffold a new connector integration across the catalog and runtime services.

## Usage

`/add-connector <connector-name>`

Where `<connector-name>` is kebab-case (e.g. `slack`, `github-issues`, `smtp-email`).

## Process

1. **Read the connector-catalog service's CLAUDE.md** to understand existing connector definitions and the config schema format.

2. **Create the catalog entry** in `repository/connector-catalog/`:
   - Add a connector definition file containing:
     - Connector name and description
     - Available triggers (events that can start a workflow)
     - Available actions (operations the connector can perform)
     - Config schema (what fields the user must provide)
     - Authentication type (API key, OAuth2, basic auth, etc.)

3. **Create the runtime adapter** in `repository/connector-runtime/`:
   - Implement the adapter as a plugin following the existing adapter pattern
   - Each trigger and action from the catalog definition needs a corresponding implementation
   - Include input/output mapping for data flowing through the workflow

4. **Add credential type** if needed — update the credential-store's supported credential types.

5. **Test the connector:**
   - Unit tests for the catalog definition validation
   - Unit tests for each trigger and action in the runtime adapter
   - Verify the connector appears in the catalog API

6. **Commit** changes to each affected submodule separately.

## Connector structure example

```
# In connector-catalog
connectors/
  slack/
    definition.json    # Triggers, actions, config schema

# In connector-runtime
adapters/
  slack/
    __init__.py
    triggers.py        # Trigger implementations
    actions.py         # Action implementations
    auth.py            # Authentication handling
    tests/
      test_triggers.py
      test_actions.py
```

## Existing connectors

Check `repository/connector-catalog/` for currently defined connectors to follow established patterns.
