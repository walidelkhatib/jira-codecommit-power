---
inclusion: always
---

# Pull Request Review Workflow

# ⚠️⚠️⚠️ BEFORE YOU START - MANDATORY REQUIREMENTS ⚠️⚠️⚠️

**Every review workflow MUST include these steps. If you skip ANY of them, the workflow is INCOMPLETE:**

- [ ] **POST review comments directly on CodeCommit PR** (Phase 4) - DO NOT just show in chat!
- [ ] **POST summary comment on CodeCommit PR** (Phase 5) - Use AWS MCP!
- [ ] **Add "kiro" label to Jira ticket** (Phase 6, Step 2) - DO NOT SKIP!
- [ ] **Post review summary comment on Jira ticket** (Phase 6, Step 3)
- [ ] **Use MCP tools, NOT CLI commands** (Throughout)

**🔴 If you complete the workflow without doing ALL of the above, you FAILED. 🔴**

**🔴 CRITICAL: ALL comments MUST be posted to CodeCommit PR using AWS MCP, not just shown in chat! 🔴**

**🔴 DO NOT ask user if you should post comments - post them automatically! 🔴**

---

## CRITICAL: Automatic PR Review

**When a user asks to review a PR (e.g., "Review PR #42" or "Review pull request 42"), you MUST execute the COMPLETE review workflow automatically.**

**This is a fully automated review workflow. Execute ALL phases in sequence:**
1. Fetch PR Details
2. Analyze Changes
3. Perform Code Review
4. Post Review Comments
5. Update PR Status
6. Update Jira Ticket (ADD "kiro" LABEL + POST COMMENT)

**Do NOT stop after any phase. Continue to completion unless there's an error.**

---

## Workflow Overview

This steering file guides the AI through automated pull request review, from fetching PR details to posting review comments on CodeCommit.

## Automatic Workflow Trigger

**When user provides a PR number or asks for review, automatically execute all phases:**

```
User input: "Review PR #42" or "Review pull request 42" or "Review PR 42"

Automatic execution:
→ Phase 1: Fetch PR Details
→ Phase 2: Analyze Changes
→ Phase 3: Perform Code Review
→ Phase 4: Post Review Comments (DIRECTLY ON CODECOMMIT PR - NOT JUST IN CHAT!)
→ Phase 5: Update PR Status (POST SUMMARY ON CODECOMMIT PR!)
→ Phase 6: Update Jira Ticket (ADD "kiro" LABEL + POST COMMENT)
→ Done!
```

**🔴🔴🔴 CRITICAL: Do NOT just show feedback in chat - you MUST post comments on the PR using AWS MCP! 🔴🔴🔴**

**Every review comment MUST be posted to CodeCommit using `post-comment-for-pull-request`. Showing feedback only in chat is a FAILURE.**

---

## Phase 1: Fetch PR Details

### Step 1: Get PR Information

**IMPORTANT: Use AWS MCP tool, not CLI commands:**

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

**Extract:**
- PR title and description
- Source and destination branches
- Author
- Current status
- Linked Jira ticket (if any)

### Step 2: Get PR Diff

**🔴 CRITICAL: Use AWS MCP `get-differences` - DO NOT use git diff! 🔴**

**Use AWS MCP tool to fetch the actual code changes:**

```
Tool: aws___call_aws
Arguments:
{
  "service": "codecommit",
  "operation": "get-differences",
  "parameters": {
    "repositoryName": "<REPO_NAME>",
    "beforeCommitSpecifier": "<DESTINATION_COMMIT>",
    "afterCommitSpecifier": "<SOURCE_COMMIT>"
  }
}
```

**Parameter order is critical:**
- `beforeCommitSpecifier` = DESTINATION commit (main branch) = old state
- `afterCommitSpecifier` = SOURCE commit (feature branch) = new state
- This shows what the feature branch ADDS to main (correct direction)

**❌ DO NOT use:**
- `git_diff` MCP tool
- `git diff` shell command
- Any other diff method

**Why:** Only AWS MCP `get-differences` with this parameter order shows the diff correctly for PR review.

**Analyze:**
- Files changed
- Lines added/removed
- Type of changes (new files, modifications, deletions)

---

## Phase 2: Analyze Changes

### Step 1: Understand Context

**Review PR description:**
- What is the PR trying to accomplish?
- What Jira ticket is it linked to?
- What are the acceptance criteria?

**Identify scope:**
- Which components are affected?
- What's the blast radius?
- Are there any breaking changes?

### Step 2: Categorize Changes

**Group changes by type:**
- **Feature code**: New functionality
- **Bug fixes**: Corrections to existing code
- **Tests**: Unit/integration tests
- **Documentation**: README, comments, docs
- **Configuration**: Config files, environment variables
- **Dependencies**: Package updates

---

## Phase 3: Perform Code Review

### Review Checklist

#### 1. Code Quality
- [ ] **Readability**: Is the code easy to understand?
- [ ] **Naming**: Are variables/functions well-named?
- [ ] **Complexity**: Is the code unnecessarily complex?
- [ ] **DRY**: Is there code duplication?
- [ ] **Comments**: Are complex sections documented?

#### 2. Functionality
- [ ] **Correctness**: Does the code do what it claims?
- [ ] **Edge cases**: Are edge cases handled?
- [ ] **Error handling**: Are errors caught and handled properly?
- [ ] **Logic**: Is the business logic sound?

#### 3. Testing
- [ ] **Test coverage**: Are there tests for new code?
- [ ] **Test quality**: Do tests actually test the functionality?
- [ ] **Edge case tests**: Are edge cases tested?
- [ ] **Integration tests**: Are integrations tested?

#### 4. Security
- [ ] **Input validation**: Is user input validated?
- [ ] **SQL injection**: Are queries parameterized?
- [ ] **XSS prevention**: Is output escaped?
- [ ] **Secrets**: Are there any hardcoded secrets?
- [ ] **Authentication**: Is auth properly implemented?
- [ ] **Authorization**: Are permissions checked?

#### 5. Performance
- [ ] **Efficiency**: Are there obvious performance issues?
- [ ] **Database queries**: Are queries optimized?
- [ ] **Caching**: Should caching be used?
- [ ] **Memory leaks**: Are resources properly cleaned up?

#### 6. Best Practices
- [ ] **Conventions**: Does it follow project conventions?
- [ ] **Patterns**: Are appropriate design patterns used?
- [ ] **SOLID principles**: Does it follow SOLID?
- [ ] **Code standards**: Follows `code-generation.md` standards?

#### 7. Documentation
- [ ] **Code comments**: Complex logic documented?
- [ ] **API docs**: Public APIs documented?
- [ ] **README**: README updated if needed?
- [ ] **Changelog**: Changes documented?

#### 8. Dependencies
- [ ] **Necessary**: Are new dependencies necessary?
- [ ] **Security**: Are dependencies secure (no known vulnerabilities)?
- [ ] **License**: Are licenses compatible?
- [ ] **Maintenance**: Are dependencies actively maintained?

---

## Phase 4: Post Review Comments

**🔴🔴🔴 STOP! READ THIS BEFORE CONTINUING 🔴🔴🔴**

**YOU MUST POST COMMENTS DIRECTLY ON THE CODECOMMIT PR USING AWS MCP!**

**❌ DO NOT:**
- Show review feedback only in chat
- Summarize findings in chat without posting to PR
- Ask user if you should post comments
- Ask user if you should just show feedback in chat
- Skip posting comments

**✅ YOU MUST:**
- Post EVERY review comment directly on the CodeCommit PR using `post-comment-for-pull-request`
- Use AWS MCP tool for EACH comment
- Post comments even if you also show them in chat

**Why this is critical:** The review MUST be visible on the PR in CodeCommit, not just in the chat session. Stakeholders need to see the feedback on the PR itself.

After analyzing the code, you MUST use AWS MCP to post comments directly on the CodeCommit PR. Showing feedback only in chat is NOT sufficient.

### Comment Structure

**For each issue found, post a comment with:**

1. **Severity Level**
   - 🔴 **Critical**: Must fix before merge (security, bugs)
   - 🟡 **Important**: Should fix before merge (code quality, performance)
   - 🔵 **Suggestion**: Nice to have (style, optimization)
   - ✅ **Praise**: Good practices worth highlighting

2. **Location**
   - File path
   - Line number(s)
   - Code snippet

3. **Issue Description**
   - What's wrong?
   - Why is it a problem?

4. **Recommendation**
   - How to fix it?
   - Code example if applicable

### Comment Format

```markdown
## 🔴 Critical: SQL Injection Vulnerability

**File**: `src/api/users.py`
**Line**: 45

**Issue**:
The user input is directly concatenated into the SQL query, making it vulnerable to SQL injection attacks.

**Current Code**:
```python
query = f"SELECT * FROM users WHERE email = '{email}'"
cursor.execute(query)
```

**Recommendation**:
Use parameterized queries to prevent SQL injection:

```python
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

**References**:
- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
```

### Posting Comments

**Use AWS MCP tool to post comments on the PR:**

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
    "content": "🤖 **Kiro Agent**\n\n## 🔴 Critical: SQL Injection Vulnerability\n\n**File**: `src/api/users.py`\n**Line**: 45\n\n**Issue**: ...\n\n**Recommendation**: ..."
  }
}
```

**Note:** ALL CodeCommit PR comments MUST start with "🤖 **Kiro Agent**" signature.

**You MUST post comments for:**
- ✅ All critical issues (must fix)
- ✅ All important issues (should fix)
- ✅ Top 3-5 suggestions (nice to have)
- ✅ 1-2 praise comments (positive feedback)

**IMPORTANT:** 
- ✅ **DO** post comments directly on the PR using AWS MCP
- ❌ **DO NOT** just show feedback in chat - this is NOT sufficient
- ❌ **DO NOT** ask user if you should post comments - just post them automatically
- ❌ **DO NOT** ask user if you should just show feedback in chat - always post to PR

**Guidelines:**
- ❌ Don't nitpick every minor style issue
- ❌ Don't post more than 10-15 comments (overwhelming)
- ✅ Balance criticism with praise

---

## Phase 5: Update PR Status (Optional)

### If Critical Issues Found

**Post summary comment:**

```markdown
🤖 **Kiro Agent**

## 🔴 Review Complete - Changes Required

I've reviewed this PR and found **3 critical issues** that must be addressed before merging:

1. SQL injection vulnerability in `users.py`
2. Missing authentication check in `api/admin.py`
3. Hardcoded API key in `config.py`

Please address these issues and I'll review again.

**Summary**:
- 🔴 Critical: 3
- 🟡 Important: 5
- 🔵 Suggestions: 7
- ✅ Good practices: 2
```

### If No Critical Issues

**Post approval comment:**

```markdown
🤖 **Kiro Agent**

## ✅ Review Complete - Looks Good!

I've reviewed this PR and found no critical issues. There are a few suggestions for improvement, but the code is ready to merge.

**Summary**:
- 🔴 Critical: 0
- 🟡 Important: 2
- 🔵 Suggestions: 4
- ✅ Good practices: 3

Great work on the error handling and test coverage! 🎉
```

**Note:** ALL PR comments MUST start with "🤖 **Kiro Agent**" signature.

---

## Phase 6: Update Jira Ticket (If Linked)

**🚨 CRITICAL: If the PR is linked to a Jira ticket, update the ticket automatically.**

**PHASE 6 HAS THREE STEPS - DO ALL:**
1. ✅ **Step 1: Extract Jira ticket ID from PR**
2. ✅ **Step 2: Add "kiro" label** (MANDATORY - DO NOT SKIP!)
3. ✅ **Step 3: Post review summary comment** (MANDATORY)

### Step 1: Extract Jira Ticket ID from PR

**From the PR title or description fetched in Phase 1, extract the Jira ticket ID:**

Common patterns:
- PR title: `[DEV-123] Add authentication`
- PR title: `DEV-123: Add authentication`
- PR description: Contains link to `https://company.atlassian.net/browse/DEV-123`

**If no Jira ticket is found, skip Phase 6.**

### Step 2: Add "kiro" Label to Ticket

**🔴🔴🔴 DO THIS NOW - BEFORE POSTING COMMENT 🔴🔴🔴**

**Add the "kiro" label to track AI-assisted work:**

**Step 2.1: Get current labels from the ticket:**
```
Tool: jira_get_issue
Arguments: { "issue_key": "<TICKET_ID>", "fields": "labels" }
```

**Step 2.2: Check if "kiro" label already exists:**
- If "kiro" is already in the labels array → Skip to Step 3 (no need to update)
- If "kiro" is NOT in the labels array → Continue to Step 2.3

**Step 2.3: Add "kiro" to the labels array:**
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
- ✅ Check if "kiro" already exists before updating (saves an API call)
- ✅ It's safe to add "kiro" even if it already exists (just include it in the array)
- ❌ **DO NOT use** `additional_fields.update.labels[{"add": "kiro"}]` - this syntax fails with Atlassian MCP

**🔴 DO NOT PROCEED TO STEP 3 UNTIL YOU'VE CHECKED/ADDED THE LABEL! 🔴**

### Step 3: Post Review Summary Comment to Jira Ticket

```
Tool: jira_add_comment
Arguments:
{
  "issueKey": "<TICKET_ID>",
  "comment": "🤖 *Kiro Agent*\n\n✅ PR Review Completed\n\nI've reviewed PR #<PR_ID> and posted <NUMBER> comments.\n\n**Review Summary:**\n- 🔴 Critical issues: <COUNT>\n- 🟡 Important issues: <COUNT>\n- 🔵 Suggestions: <COUNT>\n- ✅ Good practices: <COUNT>\n\nSee PR for detailed feedback: <PR_URL>"
}
```

**Comment format:**
- ✅ **MUST start with:** "🤖 *Kiro Agent*" signature
- ✅ Summary of review findings
- ✅ Count of issues by severity
- ✅ Link to the PR

**Why update Jira?**
- Keeps stakeholders informed that review is complete
- Maintains traceability in Jira
- Notifies watchers that PR has been reviewed
- Provides audit trail of AI-assisted reviews
- "kiro" label enables tracking and analytics

---

### ✅ Phase 6 Completion Checklist (VERIFY YOU DID ALL):
- [ ] ✅ Extracted Jira ticket ID from PR (Step 1)
- [ ] ✅ Added "kiro" label to ticket (Step 2)
- [ ] ✅ Posted review summary comment (Step 3)

**If you didn't do ALL of the above, go back and complete the missing steps!**

---

## Addressing Review Feedback

**After completing a review, if the user wants to implement the feedback:**

See `pr-patch-workflow.md` for the complete autonomous workflow to address review comments.

**Quick reference:**
- "Patch PR #42" → Automatically implements all critical and important fixes
- "Fix the [specific issue] in PR #42" → Implements only that specific fix

The patch workflow is fully autonomous and will fetch comments, implement fixes, commit, push, and post updates without requiring confirmation.

---

## Review Best Practices

### Be Constructive
- ✅ Explain WHY something is an issue
- ✅ Provide specific recommendations
- ✅ Include code examples
- ✅ Link to documentation/resources
- ✅ Acknowledge good practices

### Be Balanced
- ✅ Mix criticism with praise
- ✅ Prioritize issues (critical vs. nice-to-have)
- ✅ Don't overwhelm with too many comments
- ✅ Focus on important issues

### Be Specific
- ✅ Reference exact file and line numbers
- ✅ Quote the problematic code
- ✅ Show the recommended fix
- ✅ Explain the impact

### Be Professional
- ✅ Focus on the code, not the person
- ✅ Use objective language
- ✅ Assume good intent
- ✅ Be respectful and encouraging

---

## Example Review Workflow

### User Input
```
"Review PR #42"
```

### Automatic Execution

```
Phase 1: Fetching PR #42...
- Title: "Add OAuth2 authentication"
- Author: john.doe
- Files changed: 8
- Lines: +245, -12

Phase 2: Analyzing changes...
- Feature code: 5 files
- Tests: 2 files
- Documentation: 1 file

Phase 3: Performing code review...
- Checking code quality... ✓
- Checking functionality... ✓
- Checking tests... ⚠️ Missing edge case tests
- Checking security... 🔴 Found SQL injection vulnerability
- Checking performance... ✓
- Checking best practices... ✓
- Checking documentation... ✓

Phase 4: Posting review comments...
- Posted 1 critical issue
- Posted 2 important issues
- Posted 3 suggestions
- Posted 1 praise comment

Phase 5: Updating PR status...
- Posted summary comment

Phase 6: Updating Jira ticket DEV-123...
- Added "kiro" label
- Posted review summary comment

✅ Review complete! Posted 7 comments on PR #42. Jira ticket updated.
```

---

## REMINDER: Complete Review Execution

**When a user asks to review a PR, you MUST:**

1. ✅ **Fetch PR details and diff** (Phase 1)
2. ✅ **Analyze the changes** (Phase 2)
3. ✅ **Perform comprehensive code review** (Phase 3)
4. ✅ **Post review comments ON THE PR using AWS MCP** (Phase 4)
   - ❌ NOT just in chat!
5. ✅ **Update PR status** (Phase 5)
6. ✅ **Update Jira ticket if linked** (Phase 6) - **ALL THREE STEPS:**
   - ✅ Extract Jira ticket ID from PR (Step 1)
   - ✅ Add "kiro" label to ticket (Step 2)
   - ✅ Post review summary comment (Step 3)

**Do NOT stop after any phase.**
**Do NOT ask for confirmation between phases.**
**Do NOT just show feedback in chat - POST IT ON THE PR.**
**Execute the complete review workflow automatically.**

**This is the expected behavior. Execute the complete workflow automatically.**

---

## ✅ FINAL VERIFICATION - RUN THIS BEFORE REPORTING COMPLETION

**Before you tell the user the review is complete, verify ALL of these:**

- [ ] ✅ **Did I post review comments on the PR?** (Phase 4)
- [ ] ✅ **Did I post the summary comment on the PR?** (Phase 5)
- [ ] ✅ **Did I add the "kiro" label to the Jira ticket?** (Phase 6, Step 2)
- [ ] ✅ **Did I post the Jira comment?** (Phase 6, Step 3)
- [ ] ✅ **Did I use MCP tools instead of CLI commands?** (Throughout)

**🔴 IF ANY CHECKBOX IS UNCHECKED, GO BACK AND FIX IT NOW! 🔴**

**DO NOT report completion to the user until ALL checkboxes are checked!**
