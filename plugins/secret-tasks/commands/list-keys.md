---
description: List configured keys (private key and public key recipients)
allowed-tools: ["Read"]
---

Display the current key configuration for secret tasks in this project.

**Workflow:**

1. **Check for settings file:**
   - Look for `.claude/secret-tasks.local.md`
   - If not found, check environment variables instead

2. **If settings file exists:**
   - Read and parse YAML frontmatter
   - Display `private_key` (for decryption)
   - Display all `public_keys` (recipients who can decrypt)

3. **If no settings file:**
   - Show `SAFE_KEY_PATH` env var value (or "not set")
   - Show `SAFE_PUB_KEY` env var value (or "not set")

4. **Format output:**

   ```
   ## Secret Tasks Key Configuration

   **Private Key (for decryption):**
   ~/.safe/mykey.x25519.key

   **Public Keys (recipients who can decrypt):**
   1. ~/.safe/alice.x25519.pub
   2. ~/.safe/bob.x25519.pub
   3. /shared/team.pub

   Tasks encrypted with create-plan will be decryptable by any of the 3 recipients above.
   ```

5. **If no configuration found:**
   ```
   No key configuration found.

   To configure keys:
   - /secret-tasks:use-key [private-key-path]
   - /secret-tasks:add-recipient [public-key-path]

   Or set environment variables:
   - SAFE_KEY_PATH (private key for decryption)
   - SAFE_PUB_KEY (public key for encryption)
   ```
