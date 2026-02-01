---
name: secret-tasks
description: Use this skill for encrypted task management - generating keypairs, configuring keys, creating encrypted task plans, and decrypting tasks. Triggers on "secret tasks", "encrypted tasks", "safe CLI", "keygen", "create plan", "decrypt task", or key configuration requests.
version: 0.1.0
---

# Secret Tasks

Create and manage encrypted task files for secure multi-agent orchestration using the [safe CLI](https://github.com/grittygrease/safe).

## Quick Reference

| Action | Trigger Phrases |
|--------|-----------------|
| Generate keypair | "generate keys", "keygen", "create keypair" |
| Configure private key | "use key", "set private key", "configure decryption" |
| Add recipient | "add recipient", "add public key" |
| List keys | "list keys", "show configuration", "what keys" |
| Create plan | "create plan", "create encrypted tasks", "make secret plan" |
| Decrypt task | "decrypt task", "show task", "reveal task" |

---

## Workflows

### Generate Keypair

**When:** User wants to create new encryption keys.

**Steps:**

1. Create directory:
   ```bash
   mkdir -p ~/.safe
   ```

2. Generate keypair (use provided name or default to "secret-tasks"):
   ```bash
   safe keygen x25519 -o ~/.safe/[name]
   ```

3. Report results:
   - Private key: `~/.safe/[name].x25519.key` (keep secret)
   - Public key: `~/.safe/[name].x25519.pub` (share with team)

4. Remind user:
   - Never commit `.key` files
   - Add `~/.safe/*.key` to global gitignore
   - Share `.pub` files with collaborators

---

### Configure Private Key

**When:** User wants to set which key to use for decryption.

**Steps:**

1. Validate key exists:
   ```bash
   test -f "[path]" && echo "OK" || echo "ERROR: Key not found"
   ```

2. Create settings directory:
   ```bash
   mkdir -p .claude
   ```

3. Read existing settings from `.claude/secret-tasks.local.md` (if exists) to preserve `public_keys`

4. Write updated settings:
   ```markdown
   ---
   private_key: [path]
   public_keys:
     - [preserved existing keys]
   ---

   # Secret Tasks Configuration
   Add this file to .gitignore.
   ```

5. Confirm: Show configured key path, note it overrides `SAFE_KEY_PATH` env var

---

### Add Recipient

**When:** User wants to add a public key that can decrypt tasks.

**Steps:**

1. Validate key exists:
   ```bash
   test -f "[path]" && echo "OK" || echo "ERROR: Key not found"
   ```

2. Optionally verify it's a valid public key:
   ```bash
   safe keyinfo "[path]" 2>/dev/null | grep -q "public"
   ```

3. Create settings directory if needed:
   ```bash
   mkdir -p .claude
   ```

4. Read existing settings, check for duplicates

5. Append to `public_keys` list and write updated settings

6. Confirm: Show added key, total recipients count

---

### List Keys

**When:** User wants to see current key configuration.

**Steps:**

1. Check for `.claude/secret-tasks.local.md`

2. If found, parse YAML frontmatter and display:
   - Private key (for decryption)
   - All public keys (recipients)

3. If not found, show environment variables:
   - `SAFE_KEY_PATH` (or "not set")
   - `SAFE_PUB_KEY` (or "not set")

4. If nothing configured, show setup instructions

---

### Create Encrypted Plan

**When:** User wants to create an encrypted task plan from requirements.

**Prerequisites:** At least one public key configured.

**Steps:**

1. **Load key configuration:**
   - Check `.claude/secret-tasks.local.md` (project-local)
   - Fall back to `~/.claude/secret-tasks.local.md` (user-global)
   - Fall back to `SAFE_PUB_KEY` env var
   - If none: stop and show configuration instructions

2. **Gather requirements:**
   - Ask user to describe the feature, or
   - Read from spec file if provided

3. **Generate 10-30 tasks:**
   - Each completable in <30 minutes
   - IDs as strings: "1", "2", "3"
   - All start as "pending"
   - Use blockedBy/blocks for dependencies

4. **For each task, encrypt description:**

   With multiple recipients:
   ```bash
   echo "DESCRIPTION" | safe encrypt -i - -o - -r key1.pub -r key2.pub | base64
   ```

   Single recipient:
   ```bash
   echo "DESCRIPTION" | safe encrypt -i - -o - -r "$SAFE_PUB_KEY" | base64
   ```

5. **Create task JSON:**
   ```json
   {
     "id": "N",
     "subject": "Secret Task N",
     "description": "Decrypt the following using the safe CLI: \"BASE64\"",
     "activeForm": "Working on secret task N",
     "status": "pending",
     "blockedBy": [],
     "blocks": []
   }
   ```

6. **Write files:**
   ```bash
   mkdir -p ~/.claude/tasks/[feature-slug]
   ```
   Write each task to `~/.claude/tasks/[feature-slug]/N.json`

7. **Report:**
   - Total tasks created
   - Output directory
   - Launch command: `CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/[slug] claude`

---

### Decrypt Task

**When:** User wants to see a task's actual description, or Claude encounters encrypted task content.

**Trigger detection:** Description contains "Decrypt the following using the safe CLI:"

**Steps:**

1. **Load private key:**
   - Check `.claude/secret-tasks.local.md` for `private_key`
   - Fall back to `SAFE_KEY_PATH` env var
   - If none: stop and show configuration instructions

2. **Locate task file:**
   - If given task ID (number): search `~/.claude/tasks/*/$ID.json`
   - If given path: use directly

3. **Read and parse task JSON**

4. **Extract encrypted content** from description (base64 between quotes)

5. **Decrypt:**
   ```bash
   echo "BASE64" | base64 -d | safe decrypt -i - -o - -k "[private-key-path]"
   ```

6. **Display:**
   ```
   Task #N: Secret Task N
   Status: pending
   Blocked by: [deps]
   Blocks: [deps]

   --- Decrypted Description ---
   [actual task content]
   ```

**Error handling:**
- Wrong key: "Decryption failed - key may not match"
- File not found: "Task file not found" + list available
- No key configured: Show setup instructions

---

## Settings File Format

Location: `.claude/secret-tasks.local.md`

```yaml
---
private_key: ~/.safe/mykey.x25519.key
public_keys:
  - ~/.safe/alice.x25519.pub
  - ~/.safe/bob.x25519.pub
  - /shared/team.pub
---

# Secret Tasks Configuration
Add this file to .gitignore.
```

## Environment Variables (Fallback)

| Variable | Purpose |
|----------|---------|
| `SAFE_KEY_PATH` | Private key for decryption |
| `SAFE_PUB_KEY` | Public key for encryption (single recipient) |

Settings file takes priority over environment variables.

---

## Task File Format

```json
{
  "id": "1",
  "subject": "Secret Task 1",
  "description": "Decrypt the following using the safe CLI: \"c2FmZS1lbmNy...\"",
  "activeForm": "Working on secret task",
  "status": "pending",
  "blockedBy": [],
  "blocks": ["2", "3"]
}
```

**Visible (no key needed):** IDs, subjects, dependencies, status
**Hidden (encrypted):** Actual descriptions, file paths, implementation details

---

## Launching with Task List

```bash
CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/my-feature claude
```

Claude Code will:
- Load tasks from the directory
- Track status and dependencies
- Automatically decrypt when needed (using this skill)

---

## Troubleshooting

**"No private key configured"**
```bash
# Configure via settings (recommended)
# Tell Claude: "use key ~/.safe/mykey.x25519.key"

# Or via environment
export SAFE_KEY_PATH=~/.safe/mykey.x25519.key
```

**"Decryption failed"**
- Wrong key (doesn't match public key used for encryption)
- Corrupted content
- safe CLI not installed

**"safe: command not found"**
Install from: https://github.com/grittygrease/safe
