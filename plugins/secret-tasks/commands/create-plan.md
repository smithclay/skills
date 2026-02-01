---
description: Create encrypted task plan from requirements
argument-hint: [feature-slug]
allowed-tools: ["Read", "Write", "Bash(safe:*)", "Bash(mkdir:*)", "Bash(cat:*)", "Bash(echo:*)", "Glob", "Grep"]
---

Create an encrypted task plan for a feature. Generate 10-30 tasks with encrypted descriptions that only authorized team members can decrypt.

**Arguments:**
- `$1` = Feature slug (kebab-case, e.g., "user-auth", "api-refactor")

**Prerequisites:**
- At least one public key must be configured (settings file or env var)

**Workflow:**

1. **Load Key Configuration**

   Check for settings file in this order:
   1. **Project-local**: `<project-root>/.claude/secret-tasks.local.md` (in current working directory)
   2. **User-global**: `~/.claude/secret-tasks.local.md`

   If found, parse YAML frontmatter for `public_keys` list - these are the recipients who can decrypt the tasks.

   If no settings file found, fall back to `SAFE_PUB_KEY` environment variable.

   If neither configured, stop and instruct user:
   ```
   No public keys configured. Use one of:
   - /secret-tasks:add-recipient [pub-key-path]  (recommended)
   - export SAFE_PUB_KEY=/path/to/key.pub
   ```

2. **Gather Requirements**
   - Ask the user to describe the feature requirements, or
   - Read from a spec file if one exists (e.g., `docs/specs/$1.md` or similar)

2. **Generate Tasks**
   Create 10-30 tasks following these rules:
   - Each task completable in <30 minutes
   - IDs are string numbers: "1", "2", "3", etc.
   - All statuses start as "pending"
   - Use blockedBy/blocks for dependencies

   Task granularity guide:
   - Too coarse: "Implement authentication" (break down)
   - Too fine: "Add import statement" (merge up)
   - Just right: "Implement JWT token generation in auth/tokens.py"

3. **For Each Task**

   a. Create the full task description with:
      - Detailed description of what to do
      - Files to modify (if known)
      - Acceptance criteria / testable outcome

   b. Encrypt the description with all configured recipients:

      If using settings file with multiple public_keys:
      ```bash
      echo "FULL_DESCRIPTION_HERE" | safe encrypt -i - -o - -r key1.pub -r key2.pub -r key3.pub | base64
      ```

      Add a `-r` flag for each public key in the `public_keys` list.

      If using single SAFE_PUB_KEY env var:
      ```bash
      echo "FULL_DESCRIPTION_HERE" | safe encrypt -i - -o - -r "$SAFE_PUB_KEY" | base64
      ```

      Any of the corresponding private keys can decrypt the result.

   c. Create the task JSON with format:
      ```json
      {
        "id": "N",
        "subject": "Secret Task N",
        "description": "Decrypt the following using the safe CLI: \"ENCRYPTED_BASE64\"",
        "activeForm": "Working on secret task N",
        "status": "pending",
        "blockedBy": [],
        "blocks": []
      }
      ```

4. **Create Output Directory**
   ```bash
   mkdir -p ~/.claude/tasks/$1
   ```

5. **Write Task Files**
   Write each task to `~/.claude/tasks/$1/N.json`

**Important:**
- The `subject` field is visible and shows "Secret Task N"
- The `description` field contains encrypted content with decryption instructions
- Dependencies (blockedBy/blocks) remain visible for orchestration
- Only agents with the private key can decrypt and execute tasks

**Example encrypted description format:**
```
Decrypt the following using the safe CLI: "c2FmZS1lbmNyeXB0ZWQtY29udGVudC1oZXJl..."
```

After creating all tasks, display:
- Total tasks created
- Output directory path
- How to launch Claude Code with this task list:
  ```bash
  CLAUDE_CODE_TASK_LIST_ID=~/.claude/tasks/$1 claude
  ```
- How to decrypt a task manually: `/secret-tasks:decrypt-task 1`
