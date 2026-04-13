---
name: scaffold-services
description: Creates new Automagically service repositories from templates and adds them as git submodules. Use when adding a new microservice or when initially scaffolding the platform. Handles repo creation from svo/python-sprint-zero or svo/www-qual-is templates, template reference renaming, port assignment, Vagrantfile updates, and CLAUDE.md configuration.
disable-model-invocation: true
allowed-tools: Bash(gh *), Bash(git *), Bash(cd *), Bash(find *), Bash(fgrep *), Bash(sed *), Bash(mv *), Read, Edit, Write, Grep, Glob
---

# Scaffold Services

Creates new Automagically service repositories from templates and wires them into the platform as submodules.

## Usage

`/scaffold-services <service-name> <template> <port> <description>`

Arguments:
- `$0`: Service name in kebab-case (e.g. `notification-router`)
- `$1`: Template — either `python` (uses `svo/python-sprint-zero`) or `frontend` (uses `svo/www-qual-is`)
- `$2`: VM port number (host port will be `2` prefix, e.g. `8011` → `28011`)
- `$3`: One-line description of the service

## Steps

1. **Create the GitHub repo from template:**

```bash
gh repo create svo/automagically-$0 --template svo/$1-sprint-zero --public
```

For frontend template use `svo/www-qual-is` instead.

2. **Add as submodule:**

For Python services:
```bash
git submodule add git@github.com:svo/automagically-$0.git repository/$0
```

For frontend:
```bash
git submodule add git@github.com:svo/automagically-$0.git repository/ui
```

3. **Rename template references** (Python services only):

Derive the underscore and titlecase forms of the service name, then:

```bash
underscore=$(echo $0 | tr '-' '_')
titlecase=$(echo $0 | sed 's/-/ /g' | sed 's/\b\w/\u&/g')

# File contents
fgrep -rl python-sprint-zero . | xargs sed -i "s/python-sprint-zero/automagically-$0/g"
fgrep -rl python_sprint_zero . | xargs sed -i "s/python_sprint_zero/automagically_${underscore}/g"
fgrep -rl "Python Sprint Zero" . | xargs sed -i "s/Python Sprint Zero/Automagically ${titlecase}/g"

# File and directory names (depth-first)
find . -depth -name "*python_sprint_zero*" | while read f; do
  mv "$f" "$(echo $f | sed "s/python_sprint_zero/automagically_${underscore}/g")"
done
find . -depth -name "*python-sprint-zero*" | while read f; do
  mv "$f" "$(echo $f | sed "s/python-sprint-zero/automagically-$0/g")"
done
```

4. **Update Vagrantfile** — add the port mapping:

```ruby
config.vm.network "forwarded_port", guest: $2, host: 2$2  # $0
```

5. **Update the service's `.claude/CLAUDE.md`** — add a `## Project Purpose` section with `### Core Domain Concepts` subsection after the top-level title. Include the port assignment and service description.

6. **Commit and push** the submodule changes, then update the parent repo's submodule reference.

## Existing services

See the port assignment table in the parent [CLAUDE.md](../../CLAUDE.md) for current allocations. The next available port is 8011 (host 28011).
