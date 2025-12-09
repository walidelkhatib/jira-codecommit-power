---
inclusion: always
---

# Jira Ticket to Code Workflow

# ⚠️⚠️⚠️ BEFORE YOU START - MANDATORY REQUIREMENTS ⚠️⚠️⚠️

**Every Jira workflow MUST include these steps. If you skip ANY of them, the workflow is INCOMPLETE:**

- [ ] **Add "kiro" label to Jira ticket IMMEDIATELY** (Phase 0 - FIRST THING!) - DO NOT SKIP!
- [ ] **Use MCP tools, NOT CLI commands** (Throughout)
- [ ] **DO NOT search for repositories or regions** - Use git remote URL and AWS MCP config!
- [ ] **IF PR CREATION FAILS: Report error to user, DO NOT search for repositories!**

**🔴 If you complete the workflow without doing ALL of the above, you FAILED. 🔴**

**🔴 CRITICAL: Repository name is in git remote URL. Region is in AWS MCP config. DO NOT SEARCH! 🔴**

---

## CRITICAL: MCP Tool Usage Rules

**ALWAYS use MCP server tools. NEVER use CLI commands like `aws` or `curl`.**

### Git Operations → Use Git MCP Server (except push)
- `git_status`, `git_create_branch`, `git_checkout`, `git_add`, `git_commit`
- `git_diff_unstaged`, `git_diff_staged`, `git_log`, `git_branch`
- ❌ DO NOT use: `git status`, `git checkout -b`, `git add`, `git commit -m`, `git log`, `git branch`
- ✅ **EXCEPTION:** Use `git push` CLI command (see Phase 3)
- ⚠️ **IMPORTANT:** Always use MCP tools instead of shell commands to avoid pager issues (git log/branch requiring 'q' press)

### AWS Operations → Use AWS MCP Server
- `aws___call_aws` with service/operation/parameters
- Example: `{"service": "codecommit", "operation": "create-pull-request", "parameters": {...}}`
- ❌ DO NOT use: `aws codecommit ...` CLI commands

### Jira Operations → Use Atlassian MCP Server
- `jira_get_issue`, `jira_update_issue`, `jira_add_comment`, `jira_transition_issue`, `jira_search_issues`
- ❌ DO NOT use: curl to Jira REST API

**Why MCP tools?** Better authentication, structured responses, cross-platform compatibility.

### Jira Label Management

**⚠️ CRITICAL: The Atlassian MCP server requires using `fields.labels` which REPLACES all labels.**

**Adding Labels (e.g., "kiro"):**

1. **First, get current labels:**
   ```
   Tool: jira_get_issue
   Arguments: { "issue_key": "PROJECT-123", "fields": "labels" }
   ```

2. **Then, add new label to the array:**
   ```json
   {
     "issue_key": "PROJECT-123",
     "fields": {
       "labels": ["existing-label-1", "existing-label-2", "new-label"]
     }
   }
   ```

**If ticket has no existing labels:**
```json
{
  "issue_key": "PROJECT-123",
  "fields": {
    "labels": ["new-label"]
  }
}
```

**Removing Labels:**

**To remove all labels:**
```json
{
  "issue_key": "PROJECT-123",
  "fields": {
    "labels": []
  }
}
```

**To remove specific label(s) while keeping others:**
1. First, get current labels: `jira_get_issue` with `fields="labels"`
2. Filter out the label(s) to remove from the array
3. Set the remaining labels:
```json
{
  "issue_key": "PROJECT-123",
  "fields": {
    "labels": ["keep-this", "and-this"]
  }
}
```

**Key Points:**
- ⚠️ **ALWAYS use `fields.labels`** - it replaces all labels
- ✅ **ALWAYS fetch current labels first** before updating
- ✅ **Include all existing labels** you want to keep in the array
- ❌ **DO NOT use `additional_fields.update.labels[{"add": "name"}]`** - this syntax fails with Atlassian MCP server

---

## CRITICAL: Complete End-to-End Workflow

**When a user provides a Jira ticket ID with the "Jira" keyword (e.g., "Jira DEV-123" or "Implement Jira DEV-123"), you MUST execute the COMPLETE workflow automatically without asking for confirmation at each step.**

**This is a fully automated workflow. Execute ALL phases in sequence:**
0. Add "kiro" Label (FIRST THING!)
1. Pre-Implementation Check (ensure clean git state)
2. Ticket Analysis
3. Implementation
4. Pull Request Creation
5. Ticket Update

**Do NOT stop after any phase. Continue to completion unless there's an error.**

## Workflow Overview

This steering file guides the AI through the complete workflow from Jira ticket analysis to pull request creation and ticket updates.

## Automatic Workflow Trigger

**When user provides a ticket ID with "Jira" keyword, automatically execute all phases:**

```
User input: "Jira DEV-123" or "Implement Jira DEV-123" or "Work on Jira ticket DEV-123"

Automatic execution:
→ Phase 0: Add "kiro" Label (FIRST THING - RIGHT AFTER FETCHING TICKET!)
→ Phase 1: Pre-Implementation Check (clean git state)
→ Phase 2: Ticket Analysis
→ Phase 3: Implementation  
→ Phase 4: Pull Request Creation
→ Phase 5: Ticket Update
→ Done!
```

**Do NOT ask "Should I create a PR?" or "Should I update the ticket?" - Just do it!**

---

## Phase 0: Add "kiro" Label (MANDATORY - DO THIS FIRST!)

**🔴🔴🔴 STOP! DO THIS BEFORE ANYTHING ELSE! 🔴🔴🔴**

### Step 1: Fetch Ticket Details

```
Tool: jira_get_issue
Arguments:
{
  "issue_key": "<TICKET_ID>",
  "fields": "summary,description,labels,status,assignee,priority"
}
```

### Step 2: Add "kiro" Label IMMEDIATELY

**🔴🔴🔴 DO THIS NOW - BEFORE ANALYZING OR IMPLEMENTING ANYTHING 🔴🔴🔴**

**Add the "kiro" label RIGHT NOW:**

**First, extract existing labels from the ticket you just fetched in Step 1:**
```
From the jira_get_issue response, get the current labels array
Example: ["bug", "high-priority"] or [] if no labels
```

**Then, add "kiro" to the labels array and update:**
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

**Why do this FIRST?**
- ✅ Signals immediately that Kiro is working on this ticket
- ✅ Can't be forgotten later when focused on implementation
- ✅ Provides early tracking of AI-assisted work
- ✅ If workflow fails mid-way, label still shows Kiro attempted it

**Important:**
- ⚠️ `fields.labels` **replaces all labels** - always include existing labels in the array
- ✅ The ticket was fetched in Step 1, so you already have the existing labels
- ✅ It's safe to add "kiro" even if it already exists (just include it in the array)
- ❌ **DO NOT use** `additional_fields.update.labels[{"add": "kiro"}]` - this syntax fails with Atlassian MCP

**🔴 DO NOT PROCEED TO PHASE 1 UNTIL YOU'VE ADDED THE LABEL! 🔴**

---

## Phase 1: Pre-Implementation Check (CRITICAL)

**Before creating a new feature branch, ALWAYS ensure a clean starting state:**

### Step 1: Check for uncommitted changes
Extract and structure:
- **Functional requirements**: What needs to be built
- **Technical requirements**: How it should be built
- **Acceptance criteria**: Definition of done
- **Dependencies**: Related tickets or components

### Step 2: Identify Affected Files
Analyze the codebase to determine:
- Which files need modification
- Which new files need creation
- Which tests need updates
- Which documentation needs changes

### Step 3: Create Implementation Plan
Generate a structured plan:
```markdown
## Implementation Plan for [TICKET-ID]

### Overview
[Brief description of changes]

### Files to Modify
- `path/to/file1.py` - [reason]
- `path/to/file2.ts` - [reason]

### Files to Create
- `path/to/new-file.py` - [purpose]

### Testing Strategy
- Unit tests: [list]
- Integration tests: [list]

### Risks and Considerations
- [Any potential issues]
```

   
   Use Git MCP tool:
   ```
   Tool: git_status
   ```

### Step 2: Handle uncommitted changes (if any)

**If there are uncommitted changes:**
   - **Option A (Recommended)**: Stash them
     ```bash
     git stash push -m "WIP: Stashing before starting new ticket"
     ```
   - **Option B**: Commit them to current branch
   - **Option C**: Ask user what to do with uncommitted changes

### Step 3: Checkout and update main branch
   
   Use Git MCP tools:
   ```
   Tool: git_checkout
   Arguments: { "ref": "main" }
   ```
   
   Then pull latest changes:
   ```bash
   git pull origin main
   ```
   
   **Note**: If the default branch is not `main`, use the appropriate branch name (e.g., `master`, `develop`)

### Step 4: Verify clean state
   
   Use Git MCP tool:
   ```
   Tool: git_status
   ```
   Should show: "nothing to commit, working tree clean"

**Why this matters:**
- Prevents mixing changes from different tickets
- Ensures new branch is based on latest main, not another feature branch
- Avoids merge conflicts and confusion

**IMPORTANT:** Always use Git MCP tools (`git_status`, `git_checkout`) instead of shell commands to avoid pager issues.

---

## Phase 2: Ticket Analysis

### Step 1: Parse Requirements

### Step 1: Create Feature Branch

**Use Git MCP tool `git_create_branch`:**

```
Tool: git_create_branch
Arguments:
{
  "branch": "feature/DEV-123-add-authentication"
}
```

**Branch naming convention:** `feature/TICKET-ID-brief-description`

### Step 2: Implement Changes
Follow these principles:
- **Incremental**: Make small, logical changes
- **Testable**: Write tests alongside code
- **Documented**: Add comments for complex logic
- **Conventional**: Follow project coding standards

### Step 3: Validate Changes
Before committing:
- Run linters and formatters
- Execute unit tests
- Check for compilation errors
- Review changes with git diff

### Step 4: Commit Changes Locally
Use conventional commit format:
```
<type>(<scope>): <subject>

<body>

Refs: TICKET-ID
```

Example:
```
feat(auth): Add OAuth2 authentication

Implements OAuth2 flow with Google and GitHub providers
as specified in the requirements.

Refs: DEV-123
```

---

## Phase 4: Push and Pull Request Creation

**CRITICAL: Use git push with git-remote-codecommit to preserve commit history**

**📝 IMPORTANT: Track the repository name from Step 1 for use in Step 3 (PR creation)**

### Step 1: Ensure Remote is Configured

**Check existing remotes:**

Use shell command (git remote doesn't have pager issues):
```bash
git remote -v
```

**Save the repository name from the remote URL for later use in Step 3:**
- Format: `codecommit::<region>://<profile>@<REPO_NAME>`
- Example: `codecommit::us-east-1://profile@my-repo` → **REPO_NAME = "my-repo"**
- Or: `codecommit://<profile>@<REPO_NAME>` → **REPO_NAME = "my-repo"**

**Handle remote configuration based on what exists:**

**Case 1: No remote configured**
```bash
git remote add origin codecommit::<AWS_REGION>://<AWS_PROFILE>@<REPO_NAME>
```

**Case 2: Remote exists but uses HTTPS/SSH URL**

Check if origin uses codecommit:: format. If not, update it:

```bash
# Check current origin URL
git remote get-url origin

# If it's not codecommit::, update it
git remote set-url origin codecommit::<AWS_REGION>://<AWS_PROFILE>@<REPO_NAME>
```

**Case 3: Remote already uses codecommit::**

No action needed - remote is already configured correctly.

**URL Format:** `codecommit::<region>://<profile-name>@<repository-name>`

**Components:**
- `region`: AWS region (e.g., us-east-1, us-west-2)
- `profile-name`: AWS CLI profile name (same as configured in AWS MCP)
- `repository-name`: CodeCommit repository name (not the full ARN, just the name)

**Examples:**
```bash
# Correct format with region
git remote set-url origin codecommit::us-east-1://my-profile@my-repository
git remote set-url origin codecommit::us-west-2://demoaccount2@aws-spa-demo

# Alternative: Without region (uses default from profile)
git remote set-url origin codecommit://my-profile@my-repository

# Wrong format (old HTTPS URL)
git remote set-url origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-repository
```

**How to determine values:**
- **Region**: Same as configured in AWS MCP (e.g., us-east-1)
- **Profile**: Same as configured in AWS MCP env (e.g., demoaccount2)
- **Repository name**: Get from CodeCommit console or `aws codecommit list-repositories --profile <PROFILE>`

**Recommended format:** Include the region explicitly for clarity: `codecommit::<region>://<profile>@<repo>`

### Step 2: Push Branch to CodeCommit

Use `git push` CLI command to push the local branch:

```bash
git push origin feature/DEV-123-add-authentication
```

**Why git push CLI?**
- Preserves local commit history (author, timestamp, message)
- Uses git-remote-codecommit (no credential helper needed)
- Uses existing AWS credentials from profile
- Standard git workflow

**If push fails:**
1. Verify git-remote-codecommit is installed: `pip list | grep git-remote-codecommit`
2. Verify AWS profile has CodeCommit permissions
3. Check repository name is correct

### Step 3: Create Pull Request

**IMPORTANT: Get repository name and region from git remote URL (already configured in Step 1):**

Use Git MCP tool to get the remote URL:
```
Tool: git_status
```
Or check git config (the remote URL contains the repository name).

**Extract repository name from git remote URL:**
- Format: `codecommit::<region>://<profile>@<REPO_NAME>`
- Example: `codecommit::us-east-1://profile@my-repo` → repository name is `my-repo`
- Or format: `codecommit://<profile>@<REPO_NAME>` → repository name is `my-repo`

**Region:** The AWS MCP will automatically use the correct region from its configuration - you don't need to specify it in the API call. The region is already configured in the power settings.

**Use AWS MCP tool `aws___call_aws`:**

**🔴 CRITICAL: Keep PR descriptions simple and single-line! 🔴**

**PR Description Rules:**
- ✅ Use concise, single-line descriptions (max ~200 characters)
- ✅ Template: `"Implements [feature] with [key changes]. Resolves [TICKET-ID]"`
- ❌ **DO NOT** use multi-line strings with `\n`
- ❌ **DO NOT** use special characters (✅, ❌, etc.)
- ❌ **DO NOT** use markdown formatting (##, -, etc.)

**Why:** Complex descriptions cause AWS MCP timeouts (300s). Keep it simple!

**Good examples:**
- `"Adds search and filter functionality for messages page. Resolves DEV-123"`
- `"Implements OAuth2 authentication with JWT tokens. Resolves AUTH-456"`
- `"Fixes memory leak in data processing pipeline. Resolves BUG-789"`

```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "create-pull-request",
  "parameters": {
    "title": "[DEV-123] Add OAuth2 authentication",
    "description": "Implements OAuth2 authentication with Google and GitHub providers. Resolves DEV-123",
    "targets": [{
      "repositoryName": "<REPO_NAME_FROM_GIT_REMOTE>",
      "sourceReference": "feature/DEV-123-add-authentication",
      "destinationReference": "main"
    }]
  }
}
```

**DO NOT:**
- ❌ Run `aws codecommit list-repositories` to find the repo (you already have it!)
- ❌ Run `aws configure list` to check region (AWS MCP handles this!)
- ❌ Ask the user for repository name (extract it from git remote URL!)

**The repository name is already in your git remote URL from Step 1. Just extract it.**

**🔴🔴🔴 IF PR CREATION FAILS - READ THIS 🔴🔴🔴**

**Common failure causes:**

1. **Timeout (300s):** Description is too complex
   - ✅ **Solution:** Use simple, single-line description (see above)
   - ❌ **Don't:** Use multi-line markdown with special characters

2. **Region mismatch:** MCP region doesn't match CodeCommit repository region
   - ✅ **Solution:** Report error to user and ask them to verify region configuration
   - ❌ **Don't:** Search for repositories in different regions
   - ❌ **Don't:** Run `aws codecommit list-repositories --region <ANY_REGION>`
   - ❌ **Don't:** Assume region from API Gateway URLs or other AWS resources in the code

**If you get an error:**
1. ✅ **Report the error to the user** with the exact error message
2. ✅ **Ask the user** if the MCP region configuration matches their CodeCommit repository region
3. ✅ **Explain** that the repository might be in a different region than the MCP config
4. ✅ **Wait for user guidance** - don't try to fix it yourself

**Example error message to user:**
```
I tried to create the PR but got an error: [ERROR_MESSAGE]

This might be because:
1. The PR description was too complex (use simple single-line description), OR
2. The AWS MCP is configured for region X, but your CodeCommit repository 
   might be in a different region.

The code has been pushed successfully to the branch. Can you verify:
- What region is your CodeCommit repository in?
- Does the MCP configuration need to be updated?
```

### Step 4: Verify Branch and PR

After creating the PR, verify it was created successfully using AWS MCP tool:

```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "list-pull-requests",
  "parameters": {
    "repositoryName": "<REPO_NAME>",
    "pullRequestStatus": "OPEN"
  }
}
```

---

## Phase 5: Ticket Update (AUTOMATIC - DO NOT SKIP)

**🚨 CRITICAL: This phase is MANDATORY. Always update the Jira ticket after creating the PR.**

**Note:** The "kiro" label was already added in Phase 0, Step 2.

### Step 1: Add PR Link as Comment

```
Tool: jira_add_comment
Arguments:
{
  "issueKey": "<TICKET_ID>",
  "comment": "🤖 *Kiro Agent*\n\n✅ Pull Request Created\n\nPR #<PR_ID>: [<PR_TITLE>](<PR_URL>)\n\n**Changes:**\n- <summary of changes>\n\n**Status:** Ready for review"
}
```

**Note:** ALL Jira comments MUST start with "🤖 *Kiro Agent*" signature.

### Step 2: Transition Ticket Status
```
Transition ticket to "In Review" or appropriate status
```

### Step 3: Update Fields (Optional)
- Set "PR Link" custom field (if exists)
- Update story points (if estimated)

---

### ✅ Phase 5 Completion Checklist (VERIFY YOU DID ALL):
- [ ] ✅ Posted comment with PR link (Step 1)
- [ ] ✅ Transitioned ticket status (Step 2)
- [ ] ✅ "kiro" label was added in Phase 0, Step 2 (verify it was done)

**If you didn't do ALL of the above, go back and complete the missing steps!**

## Important Notes

### Repository Information

**🔴🔴🔴 CRITICAL: DO NOT SEARCH FOR REPOSITORIES 🔴🔴🔴**

**DO NOT ask the user for repository name or region:**
- ✅ Repository name is in the git remote URL (from Phase 3, Step 1)
- ✅ Region is configured in AWS MCP settings (automatically used)
- ❌ **DO NOT** run `aws codecommit list-repositories` to find it
- ❌ **DO NOT** run `aws codecommit list-repositories --region <ANY_REGION>`
- ❌ **DO NOT** run `aws configure list` to check region
- ❌ **DO NOT** search in different regions (us-east-2, us-west-2, etc.)
- ❌ **DO NOT** ask the user - you already have this information!
- ❌ **DO NOT** assume region from API Gateway URLs or other AWS resources in the code

**The region is ALWAYS from AWS MCP configuration (us-east-1 by default).**
**The repository name is ALWAYS in the git remote URL.**

**Extract repository name from git remote URL:**
```bash
# From: codecommit::us-east-1://profile@my-repo
# Repository name = "my-repo"

# From: codecommit://profile@my-repo  
# Repository name = "my-repo"
```

**If you search for repositories or try different regions, you are WASTING TIME and CONFUSING the workflow!**

### Autonomous Workflow
- Execute all phases without asking for confirmation
- Use information you already have (git remote, AWS MCP config)
- Don't interrupt the workflow to ask for details you can extract

---

## Error Handling

### Git Push Failures
If `git push` fails:
1. **Verify git-remote-codecommit is installed:**
   ```bash
   pip list | grep git-remote-codecommit
   # If not installed: pip install git-remote-codecommit
   ```

2. **Check AWS credentials:**
   ```bash
   aws sts get-caller-identity --profile <PROFILE_NAME>
   ```

3. **Verify CodeCommit permissions:**
   ```bash
   aws codecommit get-repository --repository-name <REPO_NAME> --profile <PROFILE_NAME>
   ```

4. **Check remote URL format:**
   ```bash
   git remote -v
   # Should show: codecommit://profile-name@repository-name
   ```

5. **Common errors:**
   - "fatal: unable to access": AWS credentials not configured
   - "fatal: repository not found": Wrong repository name or no permissions
   - "codecommit: command not found": git-remote-codecommit not installed

### PR Creation Failures

**🔴🔴🔴 CRITICAL: DO NOT SEARCH FOR REPOSITORIES WHEN PR CREATION FAILS 🔴🔴🔴**

**Common failure causes:**

**1. Timeout (300s) - Most Common**
- **Cause:** PR description is too complex (multi-line, special characters, markdown)
- **Solution:** Use simple, single-line description (max ~200 chars)
- **Example:** `"Implements OAuth2 authentication with JWT tokens. Resolves DEV-123"`
- ❌ **Don't use:** Multi-line strings with `\n`, special chars (✅, ❌), markdown formatting

**2. Region Mismatch**
- **Cause:** AWS MCP is configured for one region but CodeCommit repository is in another
- **Solution:** Report error to user and ask them to verify region configuration

**If PR creation fails, DO NOT:**
- ❌ Search for repositories in different regions
- ❌ Run `aws codecommit list-repositories --region <ANY_REGION>`
- ❌ Try to "debug" by checking other regions
- ❌ Assume region from API Gateway URLs or other AWS resources in the code
- ❌ Retry with complex multi-line descriptions

**Instead, DO THIS:**
1. ✅ **Check if description was too complex** - If so, retry with simple single-line description
2. ✅ **If still fails, report the error to the user** with the exact error message
3. ✅ **Explain** possible causes (description complexity or region mismatch)
4. ✅ **Ask the user** to verify the region configuration
5. ✅ **Wait for user guidance** - don't try to fix it yourself

**Example response:**
```
I tried to create the PR but got an error: [ERROR_MESSAGE]

This might be because:
1. The PR description was too complex (I'll retry with a simpler description), OR
2. The AWS MCP is configured for region X, but your CodeCommit repository 
   might be in a different region.

The code has been pushed successfully to the branch. Let me try with a simpler 
description first...
```

**Only if the user confirms the region is correct and simple description fails, then check:**
1. Verify branch exists in CodeCommit (use `get-branch`)
2. Check AWS permissions for CodeCommit
3. Ensure target branch exists

### Jira Update Failures
If ticket update fails:
1. Verify Jira API token is valid
2. Check ticket exists and is accessible
3. Ensure transition is valid for current status
4. Verify user has permission to update ticket

## Best Practices

### Code Quality
- Follow project's coding standards
- Write self-documenting code
- Add comments only when necessary
- Keep functions small and focused

### Testing
- Write tests before or alongside code
- Aim for high coverage of new code
- Include edge cases and error scenarios
- Run full test suite before pushing

### Communication
- Keep PR descriptions clear and detailed
- Link all related tickets and PRs
- Tag relevant reviewers
- Respond promptly to review comments

### Git Hygiene
- Keep commits atomic and logical
- Write clear commit messages
- Avoid committing generated files
- Don't commit secrets or credentials

## Checklist

Before creating PR, verify:
- [ ] All tests pass
- [ ] Code follows project conventions
- [ ] No linting errors
- [ ] Commit messages are clear
- [ ] Branch is up to date with main
- [ ] No merge conflicts
- [ ] Documentation is updated
- [ ] Acceptance criteria are met

After creating PR:
- [ ] PR description is complete
- [ ] Reviewers are assigned
- [ ] Jira ticket is updated
- [ ] CI/CD pipeline passes
- [ ] No security vulnerabilities introduced

---

## REMINDER: Complete Workflow Execution

**When a user provides a Jira ticket ID, you MUST:**

0. ✅ **Fetch ticket and add "kiro" label IMMEDIATELY** (Phase 0)
1. ✅ **Check git state and prepare clean workspace** (Phase 1)
2. ✅ **Analyze the ticket** (Phase 2)
3. ✅ **Create branch and implement code** (Phase 3)
4. ✅ **Create pull request** (Phase 4)
5. ✅ **Update Jira ticket** (Phase 5)

**Do NOT stop after any phase.**
**Do NOT ask for confirmation between phases.**
**Execute the complete workflow automatically.**

**Only stop if:**
- ❌ An error occurs (then report the error)
- ❌ User explicitly asks to stop
- ❌ Acceptance criteria cannot be met (then explain why)

**Example of correct behavior:**

```
User: "Jira DEV-123"

AI: Fetching Jira ticket DEV-123...
    Adding "kiro" label to ticket... ✓
    Checking git status...
    Stashing uncommitted changes...
    Checking out main branch...
    Pulling latest changes...
    Analyzing requirements...
    Creating branch feature/DEV-123-add-authentication...
    Implementing OAuth2 authentication...
    Running tests... ✓
    Committing changes locally...
    Pushing branch to CodeCommit...
    Creating pull request...
    Updating Jira ticket DEV-123...
    
    ✅ Complete! 
    - Branch: feature/DEV-123-add-authentication
    - PR: https://console.aws.amazon.com/codesuite/codecommit/.../pull-requests/42
    - Jira ticket updated and moved to "In Review"
```

**This is the expected behavior. Execute the complete workflow automatically.**

---

## ✅ FINAL VERIFICATION - RUN THIS BEFORE REPORTING COMPLETION

**🔴🔴🔴 STOP! Before you tell the user you're done, verify ALL of these: 🔴🔴🔴**

- [ ] ✅ **Did I add the "kiro" label to the Jira ticket?** (Phase 0, Step 2 - FIRST THING!)
- [ ] ✅ **Did I use MCP tools instead of CLI commands?** (Throughout)
- [ ] ✅ **Did I create the PR successfully?** (Phase 4)
- [ ] ✅ **Did I update the Jira ticket with PR link?** (Phase 5)

**🔴 IF ANY CHECKBOX IS UNCHECKED, GO BACK AND FIX IT NOW! 🔴**

**DO NOT report completion to the user until ALL checkboxes are checked!**

**ESPECIALLY CHECK: Did the "kiro" label get added in Phase 0, Step 2? If not, ADD IT NOW!**
