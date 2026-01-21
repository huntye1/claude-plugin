---
description: Create or switch to feature branch for development
argument-hint: [feature-name] [integration-branch]
allowed-tools: Bash(*)
model: sonnet
---

# Feature Branch Management

You are an expert at managing git branches for feature development. Your goal is to help the user ensure they are on the correct feature branch before starting work.

## Arguments

This command accepts the following arguments:

- **$1** (optional): Feature name (defaults to `WORK_FEATURE` environment variable)
- **$2** (optional): Integration branch name (defaults to `INTEGRATION_BRANCH` environment variable)

## Environment Variables

- **WORK_FEATURE**: The name of the feature being worked on (e.g., "auth_login", "video_player")
- **INTEGRATION_BRANCH**: The base release branch (e.g., "release/1.2.0")

## Logic

Follow these steps to create or switch to the correct feature branch:

### 1. Validate Environment

Check if required values are available:

- If `$1` is provided, use it as `feature_name`. Otherwise, check `WORK_FEATURE` environment variable.
- If `$2` is provided, use it as `integration_branch`. Otherwise, check `INTEGRATION_BRANCH` environment variable.
- If either value is missing, ask the user to provide it.

### 2. Construct Branch Name

- Extract the version number from `integration_branch`
  - Example: `release/1.2.0` → `1.2.0`
- Get the current date in `MMDD` format (e.g., `0121` for Jan 21st)
- Base format: `feat/wm/{version}_{feature_name}_{MMDD}`
- If branch already exists (locally or remotely), append incrementing suffix: `_1`, `_2`, `_3`, etc.
  - Example: `feat/wm/1.2.0_auth_login_0121_1`

### 3. Update Integration Branch

1. Fetch latest from origin: !`git fetch origin`
2. Update integration branch locally: !`git checkout "$integration_branch" && git pull origin "$integration_branch"`

### 4. Identify Feature Branch Name

- Start with base format: `feat/wm/{version}_{feature_name}_{MMDD}`
- If branch already exists (locally or remotely), append incrementing suffix: `_1`, `_2`, `_3`, etc.
  - Check for existence: !`git branch --list "$BRANCH_NAME"` and !`git ls-remote --heads origin "$BRANCH_NAME"`
  - Try: `feat/wm/{version}_{feature_name}_{MMDD}_1`, then `_2`, `_3`, etc.
  - For each candidate: check if it exists locally or remotely **including current branch**.
  - Stop when a unique name is found

### 5. Checkout & Create

- Create branch from integration branch: !`git checkout -b "$BRANCH_NAME" "$integration_branch"`

### 6. Confirmation

- Verify you are now on the correct branch: !`git branch --show-current`
- Report the status to the user

## Examples

### Example 1: With Arguments

**Input:**

```
/feature-branch auth_login release/1.2.0
```

**Result:** Switches to `feat/wm/1.2.0_auth_login_0121`

### Example 2: With Environment Variables

**Context:**

- `WORK_FEATURE` = "fix_bug"
- `INTEGRATION_BRANCH` = "release/5.4.0"
- Date = Jan 21st

**Input:**

```
/feature-branch
```

**Result:** Switches to `feat/wm/5.4.0_fix_bug_0121`

### Example 3: Mixed Arguments and Environment

**Context:**

- `WORK_FEATURE` = "video_player"
- `INTEGRATION_BRANCH` = "release/3.0.0"

**Input:**

```
/feature-branch new_feature release/4.0.0
```

**Result:** Switches to `feat/wm/4.0.0_new_feature_0121` (arguments override environment variables)
