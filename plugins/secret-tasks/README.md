# Secret Tasks Plugin

Generate and manage encrypted task files for secure multi-agent orchestration using the [safe CLI](https://github.com/grittygrease/safe).

## Features

- Create encrypted task plans where task descriptions are confidential
- Support for multiple recipients (any configured key can decrypt)
- Use existing keys or generate new ones
- Per-project key configuration via settings file
- Automatic decryption when working with encrypted tasks

## Prerequisites

- `safe` CLI installed and in PATH

## Quick Start

Just tell Claude what you need:

```
"Generate a keypair called myteam"
"Use key ~/.safe/myteam.x25519.key"
"Add recipient ~/.safe/alice.x25519.pub"
"Create an encrypted plan for user-auth"
"Decrypt task 1"
```

Claude will use the secret-tasks skill to handle key management, plan creation, and task decryption.

## Example Workflow

```
# 1. Generate a keypair
You: "Generate a keypair called myteam"

# 2. Configure your private key
You: "Use key ~/.safe/myteam.x25519.key for decryption"

# 3. Add recipients who can decrypt tasks
You: "Add recipient ~/.safe/myteam.x25519.pub"
You: "Add recipient ~/.safe/alice.x25519.pub"

# 4. Create an encrypted plan
You: "Create an encrypted plan called my-feature"

# 5. Decrypt a task to see its details
You: "Decrypt task 1"
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

### Option 2: Environment Variables (Fallback)

| Variable | Description |
|----------|-------------|
| `SAFE_KEY_PATH` | Path to private key (for decryption) |
| `SAFE_PUB_KEY` | Path to single public key (for encryption) |

Settings file takes priority over environment variables.

## Multiple Recipients

Tasks can be encrypted for multiple recipients. Any of the corresponding private keys can decrypt:

```
You: "Add recipient ~/.safe/alice.pub"   # Alice can decrypt
You: "Add recipient ~/.safe/bob.pub"     # Bob can decrypt
You: "Add recipient /shared/team.pub"    # Anyone with team key can decrypt
```

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
CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature claude
```

Claude Code will:
1. Load tasks from the specified directory
2. Show task status and dependencies
3. Automatically decrypt task descriptions when needed

## Installation

1. Clone or copy this plugin
2. Enable: `claude --plugin-dir /path/to/secret-tasks`
3. Tell Claude to generate keys and configure them
4. Create a plan and start working
