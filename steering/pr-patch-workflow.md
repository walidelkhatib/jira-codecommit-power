---
inclusion: always
---

# Pull Request Patch Workflow

# ⚠️⚠️⚠️ BEFORE YOU START - MANDATORY REQUIREMENTS ⚠️⚠️⚠️

**Every patch workflow MUST include these steps. If you skip ANY of them, the workflow is INCOMPLETE:**

- [ ] **Add "kiro" label to Jira ticket** (Step 8.1) - DO NOT SKIP!
- [ ] **Use MCP tools, NOT CLI commands** (Throughout)

**🔴 If you complete the workflow without doing ALL of the above, you FAILED. 🔴**

---

## CRITICAL: Automatic PR Patching

**When a user asks to patch a PR (e.g., "Patch PR #42" or "Fix the issues in PR #42"), you MUST execute the COMPLETE patch workflow automatically.**

**This is a fully autonomous workflow. Execute ALL steps automatically without waiting for confirmation.**

---

## Workflow Overview

This steering file guides the AI through automated implementation of PR review feedback, from fetching review comments to pushing fixes and posting updates on CodeCommit.

## Automatic Workflow Trigger

**When user provides a patch request, automatically execute all steps:**

```
User input: "Patch PR #42" or "Fix the issues in PR #42"

Automatic execution:
→ Step 1: Fetch PR Details and Review Comments (identifies branch)
→ Step 2: Show Implementation Plan (optional)
→ Step 3: Checkout PR Branch (using source branch from Step 1)
→ Step 4: Implement Fixes
→ Step 5: Show Changes (optional)
→ Step 6: Commit and Push Changes
→ Step 7: Post Update Comment on PR
→ Step 8: Update Jira Ticket (ADD "kiro" LABEL + POST COMMENT)
→ Done!
```

**You MAY show plan/progress for transparency, but DO NOT wait for user confirmation at any step.**

---

## Trigger Phrases

User can trigger this workflow with any of these phrases:
- "Patch PR #42" (addresses all review comments)
- "Address review comments in PR #42" (addresses all)
- "Implement feedback from PR #42" (addresses all)
- "Apply review feedback for PR #42" (addresses all)
- "Fix the issues in PR #42" (addresses all)
- "Fix the critical issues in PR #42" (only critical)
- "Fix the SQL injection in PR #42" (specific issue)
- "Fix the [specific issue] in PR #42" (specific issue)

---

## Step 1: Fetch PR Details and Review Comments

**CRITICAL: First, fetch PR details to identify the branch:**

```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "get-pull-request",
  "parameters": {
    "pullRequestId": "<PR_ID>"
  }
}
```

**Extract from PR details:**
- Source branch name (the feature branch to checkout)
- Destination branch name (usually main/master)
- Repository name
- Source commit ID
- Destination commit ID

**Then fetch review comments:**
```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "get-comments-for-pull-request",
  "parameters": {
    "pullRequestId": "<PR_ID>",
    "repositoryName": "<REPO_NAME>"
  }
}
```

**Analyze and categorize comments:**
- 🔴 Critical issues (must fix)
- 🟡 Important issues (should fix)
- 🔵 Suggestions (nice to have)

**Determine scope:**
- If user specified a specific issue (e.g., "Fix the SQL injection"), only address that
- If user said "Fix the critical issues", only address critical issues
- If user said "Patch PR" or "Address review comments", address all critical and important issues

---

## Step 2: Show Implementation Plan (Optional)

**You MAY show the user your plan for transparency, but proceed immediately without waiting:**

```markdown
Found the following issues to address in PR #42:

🔴 Critical Issues:
1. SQL injection vulnerability in `src/api/users.py` line 45
   - Will use parameterized queries

2. Hardcoded API key in `config.py` line 12
   - Will move to environment variable

🟡 Important Issues:
3. Missing error handling in `service.py` line 78
   - Will add try-catch block

4. Missing tests for edge cases
   - Will add tests for null inputs

Proceeding with implementation...
```

**DO NOT wait for confirmation. Continue immediately to Step 3.**

---

## Step 3: Checkout PR Branch

**Checkout the PR branch automatically using the source branch name from Step 1:**

**IMPORTANT: Always use Git MCP tools instead of shell commands to avoid pager issues:**

Use Git MCP tools:
```
Tool: git_checkout
Arguments: { "ref": "<SOURCE_BRANCH_NAME_FROM_PR>" }
```

Example: If PR details show source branch is "feature/MFLP-12-add-message-edit":
```
Tool: git_checkout
Arguments: { "ref": "feature/MFLP-12-add-message-edit" }
```

Then pull latest changes using shell command:
```bash
git pull origin <SOURCE_BRANCH_NAME_FROM_PR>
```

**DO NOT use shell commands like `git checkout` or `git branch`** - they may cause pager issues requiring manual 'q' press. The Git MCP server is configured with `GIT_PAGER=cat` which only applies to MCP tool calls.

**IMPORTANT:** Use the exact source branch name from the PR details fetched in Step 1. Do NOT guess the branch name.

---

## Step 4: Implement Fixes

**Implement the fixes in order of priority:**
1. Critical issues first
2. Important issues second
3. Suggestions if user specifically requested

**Follow standards:**
- Use `code-generation.md` standards
- Add tests for fixes
- Update documentation if needed
- Keep changes minimal and focused

**Implementation Guidelines:**

### For Security Issues
- SQL injection → Use parameterized queries
- XSS vulnerabilities → Escape output, validate input
- Hardcoded secrets → Move to environment variables
- Missing authentication → Add auth checks
- Missing authorization → Add permission checks

### For Code Quality Issues
- Code duplication → Extract to shared function
- Complex logic → Simplify and add comments
- Poor naming → Rename variables/functions
- Missing error handling → Add try-catch blocks

### For Testing Issues
- Missing tests → Add unit tests
- Missing edge case tests → Add edge case coverage
- Failing tests → Fix the underlying issue

### For Performance Issues
- Inefficient queries → Optimize or add indexes
- Missing caching → Add appropriate caching
- Memory leaks → Ensure proper cleanup

---

## Step 5: Show Changes (Optional)

**You MAY show what changed for transparency, but proceed immediately:**

**IMPORTANT: Always use Git MCP tools instead of shell commands to avoid pager issues:**

Use Git MCP tool:
```
Tool: git_diff_unstaged
```

**For viewing commit history, use the git_log MCP tool:**
```
Tool: git_log
Arguments: { "max_count": 5 }
```

**DO NOT use shell commands like `git log` directly** - they may cause pager issues requiring manual 'q' press. The Git MCP server is configured with `GIT_PAGER=cat` which only applies to MCP tool calls.

**Optionally present summary:**
```markdown
Changes made:
- src/api/users.py: Fixed SQL injection with parameterized queries
- config.py: Moved API key to environment variable
- service.py: Added error handling
- tests/test_users.py: Added edge case tests

Committing and pushing...
```

**DO NOT wait for confirmation. Continue immediately to Step 6.**

---

## Step 6: Commit and Push Changes

**Commit the changes automatically:**

Use Git MCP tools:
```
Tool: git_add (files: ["src/api/users.py", "config.py", "service.py", "tests/test_users.py"])
Tool: git_commit (message: "fix: address PR review comments\n\n- Fix SQL injection vulnerability in users.py\n- Move API key to environment variable\n- Add error handling in service.py\n- Add edge case tests\n\nAddresses review comments from PR #42")
```

**Commit Message Format:**

Follow `commit-standards.md` conventions:

```
fix: address PR review comments

- Fix [critical issue 1]
- Fix [critical issue 2]
- Improve [important issue 1]
- Add [missing tests]

Addresses review comments from PR #<PR_ID>
```

**Push to CodeCommit:**

Use git CLI with git-remote-codecommit, using the source branch name from Step 1:
```bash
git push origin <SOURCE_BRANCH_NAME_FROM_PR>
```

Example: `git push origin feature/MFLP-12-add-message-edit`

---

## Step 7: Post Update Comment on PR

**After successful push, post update comment:**

```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "post-comment-for-pull-request",
  "parameters": {
    "pullRequestId": "<PR_ID>",
    "repositoryName": "<REPO_NAME>",
    "beforeCommitId": "<BEFORE_COMMIT>",
    "afterCommitId": "<AFTER_COMMIT>",
    "content": "🤖 **Kiro Agent**\n\n## ✅ Review Comments Addressed\n\nI've implemented the following fixes:\n\n- 🔴 Fixed SQL injection vulnerability in `users.py`\n- 🔴 Removed hardcoded API key, now using environment variable\n- 🟡 Added error handling in `service.py`\n- 🟡 Added edge case tests\n\nAll tests passing. Ready for re-review!"
  }
}
```

**Comment format:**
- **MUST start with:** "🤖 **Kiro Agent**" signature
- List of issues addressed
- Brief description of each fix
- Test status (if applicable)
- Ready for re-review message

---

## Step 8: Update Jira Ticket (If Linked)

**🚨 CRITICAL: If the PR is linked to a Jira ticket, update the ticket automatically.**

**STEP 8 HAS TWO SUB-STEPS - DO BOTH:**
1. ✅ **Step 8.1: Add "kiro" label** (MANDATORY - DO NOT SKIP!)
2. ✅ **Step 8.2: Post comment with update** (MANDATORY)

### Step 8.1: Extract Jira Ticket ID from PR

**From the PR title or description fetched in Step 1, extract the Jira ticket ID:**

Common patterns:
- PR title: `[DEV-123] Add authentication`
- PR title: `DEV-123: Add authentication`
- PR description: Contains link to `https://company.atlassian.net/browse/DEV-123`

**If no Jira ticket is found, skip Step 8.**

### Step 8.1: Add "kiro" Label to Ticket

**Add the "kiro" label to track AI-assisted work:**

**First, get current labels from the ticket:**
```
Tool: jira_get_issue
Arguments: { "issue_key": "<TICKET_ID>", "fields": "labels" }
```

**Then, add "kiro" to the labels array (if not already present):**
```
Tool: jira_update_issue
Arguments:
{
  "issue_key": "<TICKET_ID>",
  "fields": {
    "labels": ["existing-label-1", "existing-label-2", "kiro"]
  }
}
```

**If ticket has no existing labels:**
```
Tool: jira_update_issue
Arguments:
{
  "issue_key": "<TICKET_ID>",
  "fields": {
    "labels": ["kiro"]
  }
}
```

**Important:**
- ⚠️ `fields.labels` **replaces all labels** - always include existing labels in the array
- ✅ It's safe to add "kiro" even if it already exists (just include it in the array)
- ❌ **DO NOT use** `additional_fields.update.labels[{"add": "kiro"}]` - this syntax fails with Atlassian MCP

### Step 8.2: Post Comment to Jira Ticket

```
Tool: jira_add_comment
Arguments:
{
  "issueKey": "<TICKET_ID>",
  "comment": "🤖 *Kiro Agent*\n\n✅ PR Review Comments Addressed\n\nI've updated PR #<PR_ID> with fixes for the review feedback:\n\n🔴 Fixed SQL injection vulnerability in users.py\n🔴 Removed hardcoded API key, moved to environment variable\n🟡 Added error handling in service.py\n🟡 Added edge case tests\n\nPR: https://console.aws.amazon.com/codesuite/codecommit/repositories/<REPO_NAME>/pull-requests/<PR_ID>\n\nReady for re-review!"
}
```

**Comment format:**
- ✅ **MUST start with:** "🤖 *Kiro Agent*" signature
- ✅ Summary of fixes implemented
- ✅ List of critical and important issues addressed
- ✅ Link to the PR
- ✅ Status (ready for re-review)

**Why update Jira?**
- Keeps stakeholders informed
- Maintains traceability in Jira
- Notifies watchers that PR was updated
- Provides audit trail of work done
- "kiro" label enables tracking and analytics

**DO NOT wait for confirmation. This is the final step.**

---

### ✅ Step 8 Completion Checklist (VERIFY YOU DID ALL):
- [ ] ✅ Extracted Jira ticket ID from PR
- [ ] ✅ Added "kiro" label to ticket (Step 8.1)
- [ ] ✅ Posted comment with update (Step 8.2)

**If you didn't do ALL of the above, go back and complete the missing steps!**

---

## Example: Addressing All Comments

**User Input:**
```
"Patch PR #42"
```

**Automatic Execution:**
```
Fetching PR #42 details...
- Source branch: feature/MFLP-12-add-message-edit
- Repository: my-repo

Fetching review comments...

Found 4 issues to address:
🔴 Critical: SQL injection vulnerability
🔴 Critical: Hardcoded API key
🟡 Important: Missing error handling
🟡 Important: Missing edge case tests

Proceeding with implementation...

Checking out branch feature/MFLP-12-add-message-edit...
Implementing fixes...
- Fixed SQL injection in users.py
- Moved API key to environment variable in config.py
- Added error handling in service.py
- Added edge case tests in test_users.py

Committing changes...
Pushing to CodeCommit...
Posting update comment on PR...
Updating Jira ticket MFLP-12...

✅ Done! All review comments addressed and pushed to PR #42. Jira ticket updated.
```

---

## Example: Addressing Specific Issue

**User Input:**
```
"Fix the SQL injection in PR #42"
```

**Automatic Execution:**
```
Fetching PR #42 details...
- Source branch: feature/MFLP-8-message-delete
- Repository: my-repo

Fetching review comments...

Found SQL injection issue in `src/api/users.py` line 45.

Proceeding with fix...

Checking out branch feature/MFLP-8-message-delete...
- Fixed SQL injection with parameterized queries
- Added test to verify SQL injection is prevented

Committing changes...
Pushing to CodeCommit...
Posting update comment on PR...
Updating Jira ticket MFLP-8...

✅ Done! SQL injection fixed in PR #42. Jira ticket updated.
```

---

## Important Notes

### Autonomous Execution
- ✅ Execute all steps automatically without waiting for confirmation
- ✅ You MAY show plan/progress for transparency
- ✅ Proceed immediately through all steps
- ✅ Support both "all comments" and "specific issue" targeting

### Scope Handling
- "Patch PR #42" → Address ALL critical and important issues
- "Fix the critical issues in PR #42" → Address only critical issues
- "Fix the SQL injection in PR #42" → Address only that specific issue

### MCP Tool Usage (CRITICAL)
- ✅ Use `git_checkout` instead of `git checkout` shell command
- ✅ Use `git_status` instead of `git status` shell command
- ✅ Use `git_log` instead of `git log` shell command
- ✅ Use `git_branch` instead of `git branch` shell command
- ✅ Use `git_diff_unstaged` and `git_diff_staged` MCP tools
- ✅ Use `git_add` and `git_commit` MCP tools
- ⚠️ **EXCEPTION:** Use `git push` shell command (git-remote-codecommit)
- ⚠️ **WHY:** Git MCP server has `GIT_PAGER=cat` configured to prevent pager issues

### Do NOT
- ❌ Wait for user confirmation at any step
- ❌ Ask "Should I proceed?"
- ❌ Stop between steps
- ❌ Ask for approval before committing/pushing
- ❌ Use shell commands like `git log`, `git branch`, `git checkout` (use MCP tools)

### Standards to Follow
- ✅ Follow `code-generation.md` for code standards
- ✅ Follow `commit-standards.md` for commit messages
- ✅ Add tests for all fixes
- ✅ Keep changes minimal and focused
- ✅ Post update comment after pushing

---

## REMINDER: Complete Autonomous Execution

**When a user asks to patch a PR, you MUST:**

1. ✅ **Fetch PR details to identify the source branch** (Step 1)
2. ✅ **Fetch and analyze review comments** (Step 1)
3. ✅ **Optionally show plan** (Step 2) - but don't wait
4. ✅ **Checkout the source branch from PR details** (Step 3)
5. ✅ **Implement all fixes** (Step 4)
6. ✅ **Optionally show changes** (Step 5) - but don't wait
7. ✅ **Commit and push to source branch** (Step 6)
8. ✅ **Post update comment on PR** (Step 7)
9. ✅ **Update Jira ticket if linked** (Step 8) - **BOTH SUB-STEPS:**
   - ✅ Add "kiro" label to ticket (Step 8.1)
   - ✅ Post comment with update (Step 8.2)

**Do NOT stop after any step.**
**Do NOT ask for confirmation at any point.**
**Execute the complete patch workflow automatically.**

**This is fully autonomous. Execute immediately when triggered.**

---

## ✅ FINAL VERIFICATION - RUN THIS BEFORE REPORTING COMPLETION

**Before you tell the user the workflow is complete, verify ALL of these:**

- [ ] ✅ **Did I post the PR comment?** (Step 7)
- [ ] ✅ **Did I add the "kiro" label to the Jira ticket?** (Step 8.1)
- [ ] ✅ **Did I post the Jira comment?** (Step 8.2)
- [ ] ✅ **Did I use MCP tools instead of CLI commands?** (Throughout)
- [ ] ✅ **Did I implement all the fixes?** (Step 4)

**🔴 IF ANY CHECKBOX IS UNCHECKED, GO BACK AND FIX IT NOW! 🔴**

**DO NOT report completion to the user until ALL checkboxes are checked!**
