---
description: Generate x25519 keypair for encrypting/decrypting secret tasks
argument-hint: [key-name]
allowed-tools: ["Bash(safe:*)", "Bash(mkdir:*)"]
---

Generate a new x25519 keypair for encrypting and decrypting secret task descriptions.

**Arguments:**
- `$1` = Key name (e.g., "team", "project", "alice")

**Steps:**

1. Create the `~/.safe/` directory if it doesn't exist:
   !`mkdir -p ~/.safe`

2. Generate the keypair using safe CLI:
   ```
   safe keygen x25519 -o ~/.safe/$1
   ```

3. After generation, display:
   - Path to private key: `~/.safe/$1.x25519.key`
   - Path to public key: `~/.safe/$1.x25519.pub`

4. Show the user how to set environment variables:
   ```bash
   export SAFE_KEY_PATH=~/.safe/$1.x25519.key
   export SAFE_PUB_KEY=~/.safe/$1.x25519.pub
   ```

5. Remind user:
   - Keep the `.key` file secret - never commit it
   - Share the `.pub` file with team members who need to create encrypted tasks
   - Add `~/.safe/*.key` to global gitignore

If no key name provided, use "secret-tasks" as default.
