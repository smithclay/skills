---
name: encrypted-tasks
description: Use this skill when working with encrypted task files, using the safe CLI for encryption/decryption, setting up keypairs for secret tasks, or understanding the encrypted task workflow. Triggers on queries about "secret tasks", "encrypted tasks", "safe CLI", "task encryption", "task list security", or "SAFE_KEY_PATH/SAFE_PUB_KEY environment variables".
version: 0.1.0
---

# Encrypted Tasks Workflow

This skill provides knowledge for creating and working with encrypted task files in Claude Code's multi-agent orchestration system.

## Overview

Encrypted tasks allow teams to create task lists where task descriptions are confidential. Only agents with the correct private key can decrypt and execute tasks. Task subjects and dependencies remain visible for orchestration, but the actual work details are encrypted.

## Key Configuration

### Option 1: Settings File (Recommended)

Use plugin commands to configure keys per-project:

```bash
# Set your private key for decryption
/secret-tasks:use-key ~/.safe/mykey.x25519.key

# Add public keys for recipients who can decrypt
/secret-tasks:add-recipient ~/.safe/alice.x25519.pub
/secret-tasks:add-recipient ~/.safe/bob.x25519.pub
/secret-tasks:add-recipient /shared/team.pub

# View current configuration
/secret-tasks:list-keys
```

This creates `.claude/secret-tasks.local.md` with your key configuration.

### Option 2: Environment Variables (Fallback)

| Variable | Purpose | Example |
|----------|---------|---------|
| `SAFE_KEY_PATH` | Path to private key (for decryption) | `~/.safe/team.x25519.key` |
| `SAFE_PUB_KEY` | Path to public key (single recipient) | `~/.safe/team.x25519.pub` |

Settings file takes priority over environment variables.

### Generating Keypairs

Use the safe CLI to generate an x25519 keypair:

```bash
mkdir -p ~/.safe
safe keygen x25519 -o ~/.safe/teamname
```

This creates:
- `~/.safe/teamname.x25519.key` (private - keep secret)
- `~/.safe/teamname.x25519.pub` (public - share with team)

### Setting Environment Variables

Add to your shell profile (~/.bashrc, ~/.zshrc):

```bash
export SAFE_KEY_PATH=~/.safe/teamname.x25519.key
export SAFE_PUB_KEY=~/.safe/teamname.x25519.pub
```

## Task File Format

Encrypted tasks follow this JSON format:

```json
{
  "id": "1",
  "subject": "Secret Task 1",
  "description": "Decrypt the following using the safe CLI: \"BASE64_ENCRYPTED_CONTENT\"",
  "activeForm": "Working on secret task 1",
  "status": "pending",
  "blockedBy": [],
  "blocks": ["2", "3"],
  "metadata": {
    "plan": "feature-name"
  }
}
```

### Key Points

- **subject**: Visible, generic ("Secret Task N")
- **description**: Contains encrypted content with decryption instructions
- **blockedBy/blocks**: Visible for dependency orchestration
- **status**: Managed normally (pending, in_progress, completed)

## Encryption Workflow

### Multiple Recipients

Tasks can be encrypted for multiple recipients. Any of their private keys can decrypt:

```bash
# Encrypt for multiple recipients (OR logic - any can decrypt)
safe encrypt -i - -o - -r alice.pub -r bob.pub -r team.pub
```

Configure recipients with `/secret-tasks:add-recipient` before creating plans.

### Creating Encrypted Tasks

1. Write the full task description:
   ```
   Implement JWT token validation in src/middleware/auth.ts

   Files: src/middleware/auth.ts, src/types/token.ts

   Acceptance:
   - Tokens are validated on protected routes
   - Invalid tokens return 401
   - Token payload is parsed and attached to request
   ```

2. Encrypt with all configured public keys:
   ```bash
   # With multiple recipients
   echo "DESCRIPTION" | safe encrypt -i - -o - -r alice.pub -r bob.pub -r team.pub | base64

   # Or single recipient via env var
   echo "DESCRIPTION" | safe encrypt -i - -o - -r "$SAFE_PUB_KEY" | base64
   ```

3. Create task JSON with encrypted description:
   ```json
   {
     "id": "1",
     "subject": "Secret Task 1",
     "description": "Decrypt the following using the safe CLI: \"c2FmZS1lbmNyeXB0ZWQ...\"",
     ...
   }
   ```

## Decryption Workflow

### Decrypting a Task

1. Extract the base64 content from the description
2. Decode and decrypt:
   ```bash
   echo "BASE64_CONTENT" | base64 -d | safe decrypt -i - -o - -k "$SAFE_KEY_PATH"
   ```

### Using the Plugin Commands

- **`/secret-tasks:decrypt-task 1`** - Decrypt task #1
- **`/secret-tasks:decrypt-task ~/.claude/tasks/feature/1.json`** - Decrypt by path

## Directory Structure

Encrypted task plans are stored in:

```
~/.claude/
└── tasks/
    └── feature-name/
        ├── 1.json       # Task 1 (encrypted)
        ├── 2.json       # Task 2 (encrypted)
        └── ...
```

## Launching Claude Code with a Task List

Use the `CLAUDE_CODE_TASK_LIST_ID` environment variable to launch Claude Code with a specific task list:

```bash
CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature claude
```

Claude Code will:
- Load all task JSON files from the directory
- Track task status (pending, in_progress, completed)
- Respect task dependencies (blockedBy/blocks)
- The `task-decryptor` agent will automatically decrypt descriptions when needed

## Security Model

### What's Visible (without key)

- Task IDs and subjects ("Secret Task N")
- Dependencies (blockedBy/blocks)
- Task status
- Plan metadata

### What's Hidden (encrypted)

- Actual task descriptions
- Files to modify
- Implementation details
- Acceptance criteria

### Key Distribution

- **Public key**: Share with anyone who creates encrypted tasks
- **Private key**: Only give to agents/users who should execute tasks

This enables scenarios like:
- Managers create tasks without revealing details to unauthorized viewers
- Only designated team members can decrypt and work on tasks
- CI/CD systems can see task status without accessing content

## Common Issues

### "No private key configured"

Configure your private key using either method:

```bash
# Option 1: Settings file (recommended)
/secret-tasks:use-key ~/.safe/yourkey.x25519.key

# Option 2: Environment variable
export SAFE_KEY_PATH=~/.safe/yourkey.x25519.key
```

### "Decryption failed"

Possible causes:
1. Wrong private key (doesn't match public key used for encryption)
2. Corrupted base64 content
3. safe CLI not installed

Verify key match:
```bash
safe keyinfo ~/.safe/yourkey.x25519.key
safe keyinfo ~/.safe/yourkey.x25519.pub
```

### "safe: command not found"

Install the safe CLI:
```bash
# Check if available
which safe

# If not, install from your package manager or build from source
```

## Best Practices

1. **Key Management**
   - Store private keys outside project directories
   - Never commit private keys to git
   - Use different keypairs for different security contexts

2. **Task Design**
   - Keep task descriptions self-contained
   - Include file paths in encrypted content
   - Define clear acceptance criteria

3. **Dependency Visibility**
   - Dependencies remain visible for orchestration
   - Don't put sensitive info in blockedBy/blocks arrays
   - Use task IDs as strings ("1", "2", etc.)

4. **Team Workflow**
   - Distribute public key to task creators
   - Distribute private key to task executors
   - Rotate keys periodically for security
