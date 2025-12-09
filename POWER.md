---
name: "jira-codecommit"
displayName: "Jira-CodeCommit Development"
description: "Automate development workflows from Jira tickets to CodeCommit pull requests"
author: "Walid El Khatib"
keywords: ["jira", "codecommit", "aws", "git", "workflow", "automation", "pr", "pull request", "ticket", "issue", "patch", "review", "fix", "implement"]
---

# Jira-CodeCommit Development Power

A specialized power for automating development workflows from Jira tickets to CodeCommit pull requests.

## Overview

This power enables AI-assisted development workflows that mirror GitHub Copilot's capabilities but work with AWS CodeCommit and Jira. It automates the entire development cycle from ticket analysis to PR creation.

---

## 🚀 Three Automated Workflows

This power provides **three fully automated workflows** triggered by simple commands:

### 1. Jira Ticket → Code → PR Workflow
**Trigger:** `"Jira DEV-123"` or `"Implement Jira DEV-123"`

**What it does:**
- Fetches and analyzes Jira ticket
- Adds "kiro" label to ticket
- Creates feature branch
- Implements code changes
- Commits and pushes to CodeCommit
- Creates pull request
- Updates Jira ticket with PR link

**Example:** `"Jira MFLP-10"`

---

### 2. PR Review Workflow
**Trigger:** `"Review CodeCommit PR #42"`

**What it does:**
- Fetches PR details and code diff
- Performs comprehensive code review
- **Posts review comments DIRECTLY on CodeCommit PR** (using AWS MCP)
- **Posts summary comment on CodeCommit PR** (using AWS MCP)
- Extracts Jira ticket from PR
- Adds "kiro" label to Jira ticket
- Posts review summary on Jira ticket

**Example:** `"Review CodeCommit PR 5"`

**Note:** You can try `"Review PR 5"` but including "CodeCommit" is more reliable for power activation.

**🔴 CRITICAL Requirements:**

1. **Post comments to CodeCommit PR:** You MUST use AWS MCP `post-comment-for-pull-request` to post EVERY review comment directly on the PR. DO NOT just show feedback in chat! DO NOT ask user if you should post comments - post them automatically!

2. **Use correct diff method:** You MUST use AWS MCP `get-differences` with this parameter order:
   ```
   "beforeCommitSpecifier": "<DESTINATION_COMMIT>",  // main (old state)
   "afterCommitSpecifier": "<SOURCE_COMMIT>"         // feature (new state)
   ```
   DO NOT use `git_diff` or `git diff` - they show changes in the wrong direction!

---

### 3. PR Patch Workflow
**Trigger:** `"Patch CodeCommit PR #42"`

**What it does:**
- Fetches PR review comments
- Checks out PR branch
- Implements fixes for review feedback
- Commits and pushes changes
- Posts update comment on PR
- Adds "kiro" label to Jira ticket
- Posts update on Jira ticket

**Example:** `"Patch CodeCommit PR 5"`

**Note:** You can try `"Patch PR 5"` but including "CodeCommit" is more reliable for power activation.

---

**💡 Tip:** All workflows are fully autonomous - just provide the trigger phrase and Kiro handles everything!

---

## ⚠️⚠️⚠️ CRITICAL REQUIREMENTS - READ THIS BEFORE EVERY WORKFLOW ⚠️⚠️⚠️

**Every workflow MUST include these requirements:**

### Requirement 1: Add "kiro" Label (PHASE 0 - BEFORE ANYTHING ELSE!)

1. **Fetch the ticket:**
   ```
   Tool: jira_get_issue
   Arguments: { "issue_key": "PROJECT-123" }
   ```

2. **Add "kiro" label IMMEDIATELY:**
   
   **First, get existing labels from the ticket you just fetched:**
   ```
   Extract labels from the jira_get_issue response
   ```
   
   **Then, add "kiro" to the labels array:**
   ```
   Tool: jira_update_issue
   Arguments:
   {
     "issue_key": "PROJECT-123",
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
     "issue_key": "PROJECT-123",
     "fields": {
       "labels": ["kiro"]
     }
   }
   ```

3. **Then proceed** with git operations, analysis, and implementation

**🔴 DO NOT skip step 2. The label MUST be added before any code work begins. 🔴**

**Why this matters:**
- ✅ Tracks which tickets Kiro worked on
- ✅ Can't be forgotten later when focused on implementation
- ✅ Provides immediate visibility of AI-assisted work
- ✅ If workflow fails mid-way, label still shows Kiro attempted it

**Important Notes:**
- ⚠️ `fields.labels` **replaces all labels** - always include existing labels in the array
- ✅ The ticket was fetched in step 1, so you already have the existing labels
- ✅ It's safe to add "kiro" even if it already exists (just include it in the array)

---

### Requirement 2: Use "Kiro Agent" Signature in ALL Comments

**Every comment you post (Jira or CodeCommit) MUST start with the Kiro Agent signature:**

**For Jira comments:**
```
🤖 *Kiro Agent*

[Your comment content here]
```

**For CodeCommit PR comments:**
```
🤖 **Kiro Agent**

[Your comment content here]
```

**Why this matters:**
- ✅ Distinguishes AI comments from human comments
- ✅ Provides transparency about AI-assisted work
- ✅ Makes it clear when Kiro is involved

**Examples:**

**Jira comment:**
```
Tool: jira_add_comment
Arguments:
{
  "issueKey": "PROJECT-123",
  "comment": "🤖 *Kiro Agent*\n\n✅ Pull Request Created\n\nPR #42: [Add authentication](...)"
}
```

**CodeCommit comment:**
```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "post-comment-for-pull-request",
  "parameters": {
    "content": "🤖 **Kiro Agent**\n\n## ✅ Review Comments Addressed\n\nFixed SQL injection..."
  }
}
```

**🔴 DO NOT post any comment without the signature! 🔴**

---

## Features

- **Jira Integration**: Read tickets, parse requirements, update status and comments
- **Git Operations**: Branch management, commits, and pushes
- **CodeCommit PRs**: Create and manage pull requests via AWS CLI
- **Code Generation**: AI-assisted implementation following best practices
- **Workflow Automation**: End-to-end ticket → code → PR → update workflow

## Workflow

### 0. Add "kiro" Label (FIRST THING!)
```
AUTOMATIC: When you fetch any Jira ticket, IMMEDIATELY add the "kiro" label
```
- **This happens BEFORE analysis, BEFORE implementation, BEFORE everything else**
- Use `jira_update_issue` with `additional_fields.update.labels[{"add": "kiro"}]`
- See "CRITICAL FIRST STEP" section above for exact tool call

### 1. Ticket Analysis
```
"Analyze Jira ticket PROJECT-123 and create an implementation plan"
```
- Fetches ticket details from Jira
- **Adds "kiro" label immediately** (Phase 0)
- Parses requirements and acceptance criteria
- Identifies affected files and components
- Creates implementation plan

### 2. Implementation
```
"Implement the changes for PROJECT-123"
```
- Creates feature branch
- Generates/modifies code
- Follows project conventions and best practices
- Runs tests (if configured)

### 3. PR Creation
```
"Create a PR for PROJECT-123"
```
- Commits changes locally with conventional commit messages
- Pushes branch to CodeCommit via git-remote-codecommit
- Creates pull request with ticket details
- Links PR to Jira ticket

### 4. Ticket Update
```
"Update PROJECT-123 with PR link and move to In Review"
```
- Adds PR link as comment
- Transitions ticket status
- Notifies stakeholders

## MCP Servers

### Atlassian Rovo MCP Server
- **Purpose**: Official Atlassian remote MCP server for Jira, Confluence, and Compass
- **Type**: Remote/Cloud-based (OAuth 2.1 authentication)
- **Capabilities**: 
  - **Jira**: Read issues, create/update tickets, transition workflows, bulk operations
  - **Confluence**: Search pages, create/update content, manage spaces
  - **Compass**: Query service components, create components
- **Authentication**: OAuth 2.1 (browser-based flow on first use)
- **Rate Limits**: Based on Atlassian plan (Standard: 1000/hr, Premium/Enterprise: up to 10,000/hr)

### Git Server
- **Purpose**: Local git operations
- **Capabilities**: Branch, commit, stage, diff, log, status
- **Note**: Push operations use `git push` CLI with git-remote-codecommit

### AWS MCP Server (Unified)
- **Purpose**: Comprehensive AWS operations with guided workflows
- **Type**: Remote/Managed (runs on AWS infrastructure)
- **Capabilities**: 
  - **API Execution**: 15,000+ AWS APIs including CodeCommit operations
  - **CodeCommit Operations**: Create branches, upload files, create PRs (avoids git credential issues)
  - **Agent SOPs**: Pre-built workflows for complex tasks (VPC setup, troubleshooting, etc.)
  - **Knowledge Tools**: Real-time AWS documentation, pricing, regional availability
  - **Best Practices**: AWS Well-Architected guidance
  - **Troubleshooting**: Intelligent debugging with CloudTrail/CloudWatch analysis

## Configuration

### Post-Installation Setup

**Important:** After installing this power, you must configure:

1. **Trusted shell commands** - Allow Kiro to run git commands
2. **Auto-approve MCP tools** - Allow Kiro to use MCP servers without manual approval

**Without these settings:** You'll need to manually approve every operation (git commands, fetching tickets, creating branches, committing, creating PRs, updating Jira, etc.).

**With these settings:** Simply say "Jira DEV-123" and Kiro automatically handles the entire workflow from ticket analysis to PR creation and Jira updates.

**Required trusted commands:**
- `git remote *`
- `git push *`

See the README for detailed configuration instructions.

### Required Configuration

**Atlassian Rovo MCP:**
- **Authentication**: OAuth 2.1 (browser-based)
- **First Use**: You'll be prompted to authorize via browser
- **Requirements**: 
  - Atlassian Cloud account
  - Access to Jira and/or Confluence
  - Modern browser for OAuth flow
- **No API tokens needed** - uses secure OAuth instead

**AWS:**
- `AWS_REGION`: Your AWS region (e.g., us-east-1)
- `AWS_PROFILE`: AWS CLI profile name
- AWS credentials configured via `aws configure`

**Git:**
- Git installed locally
- **git-remote-codecommit** installed: `pip install git-remote-codecommit`
- **No git credential helper needed** - git-remote-codecommit uses AWS credentials directly

## Usage Examples

### Complete Workflow (Automatic - Recommended) ✅
```
"Jira DEV-456"
or
"Implement Jira DEV-456"
or
"Work on Jira ticket DEV-456"
```

**Kiro automatically executes:**
0. ✅ **Fetches ticket and adds "kiro" label** (FIRST THING!)
1. ✅ Analyzes ticket requirements
2. ✅ Creates implementation plan
3. ✅ Creates branch
4. ✅ Implements code
5. ✅ Runs tests
6. ✅ Commits changes
7. ✅ Creates PR
8. ✅ Updates Jira ticket

**All in one command! No need to specify each step.**

**Note:** The "kiro" label is added in step 0, immediately after fetching the ticket, before any analysis or implementation work begins.

### Step-by-Step (If You Want Control)
```
1. "Fetch and analyze Jira ticket DEV-456"
2. "Create implementation plan for Jira DEV-456"
3. "Implement the changes for Jira DEV-456"
4. "Run tests and fix any issues"
5. "Create PR for Jira DEV-456 and update ticket"
```

### Individual Operations
```
# Ticket operations
"Show me all my assigned Jira tickets"
"Get details for ticket DEV-789"
"Add comment to DEV-789: Implementation in progress"

# Git operations
"Create branch feature/DEV-789-add-authentication"
"Show git status and uncommitted changes"
"Commit changes with message: feat(auth): Add OAuth2 support"

# CodeCommit operations
"Create PR from feature/DEV-789 to main"
"List open PRs in this repository"
```

## Best Practices

### Branch Naming
- Format: `feature/TICKET-ID-brief-description`
- Example: `feature/DEV-123-add-user-authentication`

### Commit Messages
- Follow Conventional Commits format
- Include ticket ID in commit body
- Example:
  ```
  feat(auth): Add OAuth2 authentication
  
  Implements OAuth2 flow for user authentication
  as specified in DEV-123.
  ```

### PR Descriptions
- Auto-generated from Jira ticket
- Includes: Title, description, acceptance criteria
- Links back to Jira ticket

### Ticket Updates
- Add PR link as comment
- Transition to "In Review" status
- Tag relevant reviewers

## Limitations

### Current Limitations
- **Manual Trigger**: You must prompt Kiro manually (cannot assign ticket directly from Jira UI)
- **Single Repository**: Works with one repository at a time
- **No Auto-Merge**: PRs must be manually reviewed and merged

### Future Enhancements
- Jira webhook integration for automatic triggering
- Multi-repository support
- Automated testing integration
- Auto-merge for approved PRs
- Slack/Teams notifications

## Comparison with GitHub Copilot

| Feature | GitHub Copilot | This Power |
|---------|---------------|------------|
| Ticket Assignment | ✅ Direct from Jira | ⚠️ Manual prompt |
| Code Generation | ✅ | ✅ |
| PR Creation | ✅ GitHub only | ✅ CodeCommit |
| Ticket Update | ✅ | ✅ |
| Multi-repo | ✅ | ⚠️ Single repo |
| Git Provider | GitHub only | Any (CodeCommit, GitHub, GitLab) |

## Troubleshooting

### PR Creation Failures (AWS MCP Timeout)

**Problem:** `create-pull-request` times out (300s timeout) or hangs

**Cause:** Complex multi-line descriptions with special characters cause the AWS MCP server to timeout

**Solution:** Keep PR descriptions simple and single-line

✅ **Good:**
```
"Adds search and filter functionality for messages page"
"Implements OAuth2 authentication with JWT tokens"
"Fixes memory leak in data processing pipeline"
```

❌ **Bad:**
```
"## Summary\n\nImplements search...\n\n## Changes\n\n- Added...\n- Added...\n\n## Acceptance Criteria\n\n- ✅ Met..."
```

**Best Practice:**
- Use concise, single-line descriptions (max ~200 characters)
- Avoid multi-line strings with `\n`
- Avoid special characters (✅, ❌, etc.)
- Avoid markdown formatting in the description parameter
- Template: `"Implements [feature] with [key changes]. Resolves [TICKET-ID]"`

**Workaround:** Create PR with simple description, then update it manually via AWS Console if detailed description is needed.

### Atlassian Rovo MCP Connection Issues

**First-time setup:**
1. Run any Jira command in Kiro
2. You'll be prompted to authorize via browser
3. Complete OAuth flow
4. Return to Kiro - connection established

**Common issues:**
- **"Your site admin must authorize this app"**: First user needs both Jira and Confluence access
- **OAuth timeout**: Complete the browser flow within 5 minutes
- **Rate limits**: Check your Atlassian plan limits (Standard: 1000/hr, Premium: up to 10,000/hr)
- **Permissions**: Ensure you have access to the Jira projects/Confluence spaces you're querying

### CodeCommit Access Issues
```bash
# Test AWS credentials
aws sts get-caller-identity

# Test CodeCommit access
aws codecommit list-repositories --region us-east-1

# Test specific repository
aws codecommit get-repository --repository-name YOUR_REPO --region us-east-1
```

**Common issues:**
- **403 Forbidden**: Check IAM permissions for CodeCommit
- **Repository not found**: Verify repository name and region
- **Invalid credentials**: Run `aws configure` to set up credentials

### Git MCP Issues
- Ensure repository path is correct
- Verify local git repository is initialized
- Check AWS credentials have CodeCommit permissions

## Support

For issues or questions:
- Check AWS CodeCommit documentation
- Verify Jira API token permissions
- Ensure git credential helper is configured
- Test AWS CLI access to CodeCommit
