---
name: /create-xray-tests
argument-hint: '[JIRA-TICKET]'
description: Generate test cases in X-Ray from acceptance criteria
mode: single-agent
dependencies:
  - jira CLI (ankitpokhrel/jira-cli)
  - X-Ray for JIRA (test management plugin)
---

# Create X-Ray Test Cases

## Purpose
Generate test cases in X-Ray (JIRA test management) from acceptance criteria. Supports both BDD (Gherkin) and Manual Test Steps formats.

## Prerequisites

Check for JIRA CLI and authentication:
```bash
# Check if JIRA CLI is installed and authenticated
if which jira >/dev/null 2>&1; then
  if jira me >/dev/null 2>&1; then
    echo "✅ JIRA CLI ready"
  else
    echo "❌ JIRA CLI not authenticated"
    echo "Run: /setup-qa-assistant"
    exit 1
  fi
else
  echo "❌ JIRA CLI not found"
  echo "Run: /setup-qa-assistant"
  exit 1
fi
```

## Workflow

### Step 1: Get Ticket ID

Ask user: "What's your JIRA ticket number? (e.g., EPS-1234)"

Store as `$TICKET`

### Step 2: Load Acceptance Criteria

Read AC from: `.ac-verification/$TICKET/ac-checklist.md`

If file doesn't exist:
```
❌ No acceptance criteria found for $TICKET

Please run first:
  /collect-ac $TICKET

This will gather ACs from JIRA, Confluence, and Figma.
```

Display ACs found:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Acceptance Criteria Found
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. User can request email verification code
2. System sends verification email within 1 minute
3. Code expires after 15 minutes
4. User sees success message after verification
5. Invalid codes show appropriate error messages

Total: 5 acceptance criteria
```

### Step 3: Ask About X-Ray Test Creation

Ask user: "Would you like to create test cases in X-Ray?"

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 X-Ray Test Case Generation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Create test cases in X-Ray for these acceptance criteria?

[Y] Yes - Generate X-Ray test cases
[n] No - Skip (continue with regular workflow)
```

**If user says "No":**
- Skip test case generation
- Exit command
- User continues with next step in main workflow (e.g., `/generate-e2e-tests`)

**If user says "Yes":**
- Proceed to Step 4

### Step 4: Select Test Case Format

Ask user to choose test case format:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Select Test Case Format
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Which format do you want for test cases?

1. 🥒 BDD (Gherkin)
   Example:
   Given user is on login page
   When user enters valid credentials
   Then user should see dashboard

2. 📋 Manual Test Steps
   Example:
   Step 1: Login to app
   Step 2: Click on Menu
   Step 3: Verify everything displays correctly
   Expected: All menu items visible

Choose format (1 or 2):
```

Store choice as `$FORMAT = "bdd" or "manual"`

### Step 5: Select JIRA Project

Ask user: "Which JIRA project should test cases be created in?"

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Select JIRA Project
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Enter JIRA project key (e.g., EPS, QA, TEST):
```

**Validate project exists:**
```bash
# Check if project exists
jira project list --plain | grep "^$PROJECT_KEY"

if [ $? -ne 0 ]; then
  echo "❌ Project $PROJECT_KEY not found"
  echo "Available projects:"
  jira project list --plain
  exit 1
fi
```

Store as `$PROJECT`

### Step 6: Generate Test Cases

For each acceptance criterion, generate a test case based on selected format.

**For BDD Format:**

Convert AC to Gherkin format:

```gherkin
Feature: [Feature name from JIRA ticket]

Scenario: [AC description]
  Given [precondition]
  When [action]
  Then [expected outcome]
```

**Example:**

AC: "User can request email verification code"

Converts to:
```gherkin
Scenario: User can request email verification code
  Given user is logged into the application
  When user clicks "Send Verification Code" button
  Then verification email is sent to user's registered email
  And user sees "Verification code sent" message
```

**For Manual Test Steps Format:**

Convert AC to step-by-step manual test:

```
Test Steps:
1. [Action 1]
2. [Action 2]
3. [Action 3]

Expected Result:
- [Expected outcome]

Preconditions:
- [Setup requirements]
```

**Example:**

AC: "User can request email verification code"

Converts to:
```
Test Steps:
1. Login to the application with valid credentials
2. Navigate to Profile Settings
3. Click "Send Verification Code" button
4. Check email inbox

Expected Result:
- Verification email received within 1 minute
- Email contains 6-digit verification code
- "Verification code sent" message displayed in app

Preconditions:
- User has registered account
- User email is verified
```

### Step 7: Display Generated Test Cases

Show all generated test cases to user:

**For BDD Format:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 Generated X-Ray Test Cases (BDD Format)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test Case 1: Verify email verification code request
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Feature: Email Verification

Scenario: User can request email verification code
  Given user is logged into the application
  When user clicks "Send Verification Code" button
  Then verification email is sent to user's registered email
  And user sees "Verification code sent" message

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test Case 2: Verify verification email delivery time
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Feature: Email Verification

Scenario: System sends verification email within 1 minute
  Given user has requested verification code
  When system processes the request
  Then verification email is delivered within 1 minute
  And email contains valid 6-digit code

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[... continue for all ACs ...]

Total: 5 test cases generated
```

**For Manual Test Steps Format:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 Generated X-Ray Test Cases (Manual Format)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test Case 1: Verify email verification code request
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Preconditions:
- User has registered account
- User is logged out

Test Steps:
1. Navigate to login page
2. Enter valid credentials and login
3. Navigate to Profile Settings
4. Click "Send Verification Code" button
5. Check email inbox

Expected Result:
- "Verification code sent" message displayed
- Verification email received within 1 minute
- Email contains 6-digit verification code

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test Case 2: Verify verification email delivery time
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Preconditions:
- User has registered account
- User is logged in

Test Steps:
1. Click "Send Verification Code" button
2. Note the timestamp
3. Check email inbox
4. Note email delivery timestamp
5. Calculate time difference

Expected Result:
- Email delivered within 1 minute of request
- Email contains valid 6-digit code
- Code is unique per request

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[... continue for all ACs ...]

Total: 5 test cases generated
```

### Step 8: Request Approval or Changes

Ask user for approval:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Review Test Cases
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Would you like to:

1. ✅ Approve - Create these test cases in X-Ray
2. ✏️ Edit - Make changes to specific test cases
3. 🔄 Regenerate - Regenerate all with different approach
4. ❌ Cancel - Don't create test cases

Choose option (1-4):
```

**If user selects "1. Approve":**
- Proceed to Step 9

**If user selects "2. Edit":**
```
Which test case do you want to edit? (1-5):
> 2

Current test case:
[Show test case 2]

What changes would you like?
> [User provides feedback]

Updated test case:
[Show updated version]

More changes needed? (Y/n)
> n

[Return to approval prompt]
```

**If user selects "3. Regenerate":**
```
What should be different in the regeneration?
> [User provides guidance]

Regenerating test cases...
[Show new test cases]
[Return to approval prompt]
```

**If user selects "4. Cancel":**
- Exit command
- Don't create any test cases

### Step 9: Create X-Ray Test Issues in JIRA

For each approved test case, create an X-Ray Test issue in JIRA.

**X-Ray Test Issue Structure:**

```json
{
  "project": "$PROJECT",
  "issuetype": "Test",
  "summary": "[Test case title]",
  "description": "[Full test case content]",
  "customfield_xray_test_type": "Manual" or "Cucumber",
  "customfield_xray_steps": [...] (for Manual)
  "customfield_xray_scenario": "..." (for BDD)
}
```

**For BDD Format (Cucumber):**
```bash
jira issue create \
  --type="Test" \
  --project="$PROJECT" \
  --summary="[Scenario name]" \
  --custom="Test Type=Cucumber" \
  --custom="Gherkin=$GHERKIN_CONTENT" \
  --links="$TICKET"
```

**For Manual Test Steps Format:**
```bash
jira issue create \
  --type="Test" \
  --project="$PROJECT" \
  --summary="[Test name]" \
  --custom="Test Type=Manual" \
  --custom="Test Steps=$STEPS_JSON" \
  --links="$TICKET"
```

**Manual Test Steps JSON Format:**
```json
{
  "steps": [
    {
      "index": 1,
      "action": "Login to the application with valid credentials",
      "data": "",
      "result": ""
    },
    {
      "index": 2,
      "action": "Navigate to Profile Settings",
      "data": "",
      "result": ""
    },
    {
      "index": 3,
      "action": "Click 'Send Verification Code' button",
      "data": "",
      "result": "Verification code sent message displayed"
    }
  ]
}
```

**Link test to original ticket:**
```bash
# Link each test back to the original story/task
jira issue link "$TEST_KEY" "$TICKET" "Tests"
```

### Step 10: Display Summary

Show summary of created test cases:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 X-Ray Test Cases Created Successfully!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Created 5 test cases in project: EPS

Test Cases:
  ✅ EPS-5678 - Verify email verification code request
  ✅ EPS-5679 - Verify verification email delivery time
  ✅ EPS-5680 - Verify code expiration after 15 minutes
  ✅ EPS-5681 - Verify success message after verification
  ✅ EPS-5682 - Verify invalid code error messages

Format: Manual Test Steps
Linked to: EPS-1234

View in JIRA:
  https://yourcompany.atlassian.net/issues/?jql=issue%20in%20(EPS-5678%2CEPS-5679%2CEPS-5680%2CEPS-5681%2CEPS-5682)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 Next Steps:

1. Review test cases in X-Ray
2. Execute tests manually or with automation
3. Link test executions to your test plan
4. Continue with implementation: /generate-e2e-tests EPS-1234

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 11: Save Test Case Mapping

Save mapping of ACs to test cases:

Create: `.ac-verification/$TICKET/xray-test-mapping.json`

```json
{
  "ticket": "EPS-1234",
  "project": "EPS",
  "format": "manual",
  "created": "2026-03-24T19:45:00Z",
  "testCases": [
    {
      "ac": "User can request email verification code",
      "testKey": "EPS-5678",
      "testSummary": "Verify email verification code request",
      "status": "created"
    },
    {
      "ac": "System sends verification email within 1 minute",
      "testKey": "EPS-5679",
      "testSummary": "Verify verification email delivery time",
      "status": "created"
    }
  ]
}
```

## Error Handling

| Issue | Solution |
|-------|----------|
| JIRA CLI not authenticated | Show: "Run /setup-qa-assistant to configure JIRA" |
| X-Ray not installed in JIRA | Show: "X-Ray plugin not detected. Contact JIRA admin." |
| Project doesn't exist | Show available projects and ask to select again |
| Test creation fails | Show error and ask to retry or skip that test |
| AC file not found | Guide user to run /collect-ac first |

## Integration with Workflow

This command fits into the main workflow after AC collection:

```bash
# 1️⃣ Collect ACs
/collect-ac EPS-1234

# 2️⃣ (Optional) Add visual ACs from Figma
/figma-ac-extractor https://figma.com/design/xyz

# ⭐ NEW: Create X-Ray test cases
/create-xray-tests EPS-1234
→ Creates formal test cases in X-Ray

# 3️⃣ Generate automated tests
/generate-e2e-tests EPS-1234

# 4️⃣ Verify ACs
/verify-ac EPS-1234

# 5️⃣ Post to JIRA
/post-to-jira EPS-1234
```

## Usage

Run this command:
- After collecting acceptance criteria
- Before generating automated tests
- When you need formal test cases in X-Ray
- When QA team requires test case documentation

## Output

Files created:
- `.ac-verification/$TICKET/xray-test-mapping.json` - Mapping of ACs to test cases
- X-Ray Test issues in JIRA (linked to original ticket)
