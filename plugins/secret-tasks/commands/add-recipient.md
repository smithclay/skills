---
description: Add a public key that can decrypt tasks (multiple recipients supported)
argument-hint: [pub-key-path]
allowed-tools: ["Read", "Write", "Bash(test:*)", "Bash(mkdir:*)", "Bash(safe:*)"]
---

Add a public key to the list of recipients who can decrypt task descriptions. Tasks encrypted with multiple recipients can be decrypted by ANY of the corresponding private keys.

**Arguments:**
- `$1` = Path to public key file (e.g., `~/.safe/alice.x25519.pub`, `/path/to/team.pub`)

**Workflow:**

1. **Validate the key exists and is a public key:**
   ```bash
   test -f "$1" && echo "OK" || echo "ERROR: Key file not found"
   ```

   Optionally verify it's a valid public key:
   ```bash
   safe keyinfo "$1" 2>/dev/null | grep -q "public" && echo "Valid public key" || echo "Warning: May not be a valid public key"
   ```

2. **Create settings directory if needed:**
   ```bash
   mkdir -p .claude
   ```

3. **Read existing settings** from `.claude/secret-tasks.local.md`:
   - Parse YAML frontmatter
   - Get existing `public_keys` list (or create empty list)
   - Get existing `private_key` (preserve it)

4. **Check for duplicates:**
   - If the key path is already in `public_keys`, inform user and skip

5. **Append to public_keys list** and write updated settings:

   ```markdown
   ---
   private_key: [preserved from existing]
   public_keys:
     - [existing keys]
     - $1
   ---

   # Secret Tasks Configuration
   ...
   ```

6. **Confirm to user:**
   - Show the added public key
   - Show total number of recipients
   - Explain: "Tasks created with /secret-tasks:create-plan will be decryptable by any of these keys"

**Example Usage:**

```
/secret-tasks:add-recipient ~/.safe/alice.x25519.pub
/secret-tasks:add-recipient ~/.safe/bob.x25519.pub
/secret-tasks:add-recipient /shared/team-key.pub
```

After adding 3 recipients, tasks will be encrypted with:
```bash
safe encrypt -r ~/.safe/alice.x25519.pub -r ~/.safe/bob.x25519.pub -r /shared/team-key.pub
```

**Note:** At least one public key must be configured before using `/secret-tasks:create-plan`.
