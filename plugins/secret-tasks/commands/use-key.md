---
description: Configure the private key to use for decrypting tasks
argument-hint: [key-path]
allowed-tools: ["Read", "Write", "Bash(test:*)", "Bash(mkdir:*)"]
---

Configure which private key to use for decrypting secret tasks in this project.

**Arguments:**
- `$1` = Path to private key file (e.g., `~/.safe/team.x25519.key`, `/path/to/existing.key`)

**Workflow:**

1. **Validate the key exists:**
   ```bash
   test -f "$1" && echo "OK" || echo "ERROR: Key file not found"
   ```

2. **Create settings directory if needed:**
   ```bash
   mkdir -p .claude
   ```

3. **Read existing settings** (if `.claude/secret-tasks.local.md` exists):
   - Parse YAML frontmatter to preserve existing `public_keys` list
   - If no settings file exists, create new one

4. **Update settings file** at `.claude/secret-tasks.local.md`:

   ```markdown
   ---
   private_key: $1
   public_keys:
     # existing keys preserved here
   ---

   # Secret Tasks Configuration

   This file stores key configuration for the secret-tasks plugin.
   Add this file to .gitignore to keep key paths private.
   ```

5. **Confirm to user:**
   - Show the configured private key path
   - Remind them this overrides `SAFE_KEY_PATH` environment variable for this project
   - Note: Add `.claude/secret-tasks.local.md` to `.gitignore`

**Settings File Format:**

```yaml
---
private_key: ~/.safe/mykey.x25519.key
public_keys:
  - ~/.safe/alice.x25519.pub
  - ~/.safe/bob.x25519.pub
---
```

**Priority Order:**
1. Settings file (`.claude/secret-tasks.local.md`) - highest priority
2. Environment variable (`SAFE_KEY_PATH`) - fallback
