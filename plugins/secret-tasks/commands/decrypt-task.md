---
description: Decrypt and display a secret task's description
argument-hint: [task-id-or-path]
allowed-tools: ["Read", "Bash(safe:*)", "Bash(base64:*)", "Bash(cat:*)", "Bash(echo:*)", "Glob"]
---

Decrypt and display the full description of an encrypted task.

**Arguments:**
- `$1` = Task ID (e.g., "1", "5") or full path to task file

**Prerequisites:**
- A private key must be configured (settings file or env var)

**Workflow:**

1. **Load Private Key Configuration**

   First, check for settings file at `.claude/secret-tasks.local.md` then `~/.claude/secret-tasks.local.md`:
   - If exists, parse YAML frontmatter for `private_key` field
   - Use this path for decryption

   If no settings file or no `private_key` field, fall back to `SAFE_KEY_PATH` environment variable.

   If neither configured, stop and instruct user:
   ```
   No private key configured. Use one of:
   - /secret-tasks:use-key [private-key-path]  (recommended)
   - export SAFE_KEY_PATH=/path/to/key.key
   ```

2. **Locate the Task File**

   If `$1` is a number (task ID):
   - Search for task in `~/.claude/tasks/*/$1.json`
   - If multiple matches, list them and ask user to specify

   If `$1` is a path:
   - Use the path directly

2. **Read the Task File**
   Read the JSON file and parse the `description` field.

3. **Extract Encrypted Content**
   The description format is:
   ```
   Decrypt the following using the safe CLI: "BASE64_ENCRYPTED_CONTENT"
   ```

   Extract the base64-encoded encrypted content from between the quotes.

4. **Decrypt the Content**

   Using the private key path from step 1:
   ```bash
   echo "BASE64_CONTENT" | base64 -d | safe decrypt -i - -o - -k "/path/to/private.key"
   ```

5. **Display Results**
   Show:
   - Task ID and subject
   - **Decrypted description** (the actual task details)
   - Dependencies (blockedBy, blocks)
   - Status
   - Metadata

**Example Output:**
```
Task #3: Secret Task 3
Status: pending
Blocked by: [1, 2]
Blocks: [4, 5]

--- Decrypted Description ---
Implement the user authentication middleware in src/middleware/auth.ts

Files: src/middleware/auth.ts, src/types/user.ts

Acceptance:
- JWT tokens are validated on protected routes
- Invalid tokens return 401 response
- User object is attached to request
---------------------------------
```

**Error Handling:**
- If decryption fails, suggest checking SAFE_KEY_PATH
- If task file not found, list available tasks
- If multiple plans exist, show options
