# Secret Tasks Plugin

Generate and manage encrypted task files for secure multi-agent orchestration using the [safe CLI](https://github.com/grittygrease/safe).

## Features

- Create encrypted task plans where task descriptions are confidential
- Support for multiple recipients (any configured key can decrypt)
- Use existing keys or generate new ones
- Per-project key configuration via settings file
- Auto-decryption agent for seamless task execution

## Prerequisites

- `safe` CLI installed and in PATH

## Quick Start

```bash
# 1. Generate a keypair (or use existing keys)
/secret-tasks:keygen myteam

# 2. Configure your private key
/secret-tasks:use-key ~/.safe/myteam.x25519.key

# 3. Add recipients who can decrypt tasks
/secret-tasks:add-recipient ~/.safe/myteam.x25519.pub
/secret-tasks:add-recipient ~/.safe/alice.x25519.pub

# 4. Create an encrypted plan
/secret-tasks:create-plan my-feature

# 5. Decrypt a task to see its details
/secret-tasks:decrypt-task 1
```

## Commands

### `/secret-tasks:keygen`

Generate a new x25519 keypair.

```
/secret-tasks:keygen teamname
```

Creates `~/.safe/teamname.x25519.key` (private) and `~/.safe/teamname.x25519.pub` (public).

### `/secret-tasks:use-key`

Configure which private key to use for decryption in this project.

```
/secret-tasks:use-key ~/.safe/mykey.x25519.key
/secret-tasks:use-key /path/to/existing.key
```

### `/secret-tasks:add-recipient`

Add a public key to the list of recipients who can decrypt tasks.

```
/secret-tasks:add-recipient ~/.safe/alice.x25519.pub
/secret-tasks:add-recipient ~/.safe/bob.x25519.pub
```

Tasks are encrypted for ALL configured recipients. Any of them can decrypt.

### `/secret-tasks:list-keys`

Show current key configuration.

```
/secret-tasks:list-keys
```

### `/secret-tasks:create-plan`

Create an encrypted task plan from requirements.

```
/secret-tasks:create-plan my-feature
```

Generates 10-30 tasks in `~/.claude/tasks/my-feature/` with encrypted descriptions.

### `/secret-tasks:decrypt-task`

Decrypt and display a specific task's description.

```
/secret-tasks:decrypt-task 1
/secret-tasks:decrypt-task ~/.claude/tasks/my-feature/1.json
```

## Key Configuration

### Option 1: Settings File (Recommended)

The plugin stores key configuration in `.claude/secret-tasks.local.md`:

```yaml
---
private_key: ~/.safe/mykey.x25519.key
public_keys:
  - ~/.safe/alice.x25519.pub
  - ~/.safe/bob.x25519.pub
  - /shared/team.pub
---
```

Use `/secret-tasks:use-key` and `/secret-tasks:add-recipient` to manage this file.

### Option 2: Environment Variables (Fallback)

| Variable | Description |
|----------|-------------|
| `SAFE_KEY_PATH` | Path to private key (for decryption) |
| `SAFE_PUB_KEY` | Path to single public key (for encryption) |

Settings file takes priority over environment variables.

## Multiple Recipients

Tasks can be encrypted for multiple recipients. Any of the corresponding private keys can decrypt:

```
/secret-tasks:add-recipient ~/.safe/alice.pub   # Alice can decrypt
/secret-tasks:add-recipient ~/.safe/bob.pub     # Bob can decrypt
/secret-tasks:add-recipient /shared/team.pub    # Anyone with team key can decrypt
```

This uses safe's multi-recipient encryption: `safe encrypt -r alice.pub -r bob.pub -r team.pub`

## Task File Format

```json
{
  "id": "1",
  "subject": "Secret Task 1",
  "description": "Decrypt the following using the safe CLI: \"<encrypted-base64>\"",
  "activeForm": "Working on secret task",
  "status": "pending",
  "blockedBy": [],
  "blocks": ["2", "3"]
}
```

## Security Model

**Visible (without key):**
- Task IDs and subjects ("Secret Task N")
- Dependencies (blockedBy/blocks)
- Task status and metadata

**Hidden (encrypted):**
- Actual task descriptions
- Files to modify
- Implementation details
- Acceptance criteria

## Running Claude Code with a Task List

After creating an encrypted plan, launch Claude Code with the task list:

```bash
# Set the task list directory and launch Claude Code
CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature claude

# Or export it first
export CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature
claude
```

Claude Code will:
1. Load tasks from the specified directory
2. Show task status and dependencies
3. Allow agents to claim and work on tasks
4. The `task-decryptor` agent will automatically decrypt task descriptions when needed

### Combining with Key Configuration

```bash
# Full launch command with keys and task list
SAFE_KEY_PATH=~/.safe/mykey.x25519.key \
CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature \
claude --plugin-dir /path/to/secret-tasks
```

Or if using the settings file approach, just set the task list:

```bash
CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature claude
```

## Installation

1. Clone or copy this plugin
2. Enable: `claude --plugin-dir /path/to/secret-tasks`
3. Configure keys using the commands above
4. Create a plan: `/secret-tasks:create-plan my-feature`
5. Launch with task list: `CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature claude`
