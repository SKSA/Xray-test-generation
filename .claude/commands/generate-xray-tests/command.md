---
name: /generate-xray-tests
argument-hint: 'TICKET-123 [--format=bdd|manual] [--dry-run]'
description: Generate X-Ray test cases from JIRA ticket acceptance criteria
mode: single-agent
dependencies:
  - jira CLI (required)
  - X-Ray for JIRA (required)
---

# Generate X-Ray Test Cases from Acceptance Criteria

## Purpose
This command generates X-Ray test cases from existing acceptance criteria. It supports both BDD (Gherkin) and Manual test case formats, and can create the test cases directly in X-Ray or output them for review.

**Prerequisites:** Acceptance criteria should already be extracted and available. Use `/collect-ac TICKET-123` first if you need to extract ACs from the JIRA ticket.

## Prerequisites Check

### Phase 0: Environment and Setup Validation

#### 1. JIRA CLI Setup
```bash
# Check if JIRA CLI is installed
if ! command -v jira &> /dev/null; then
    echo "❌ JIRA CLI not found. Install with:"
    echo "   macOS: brew install ankitpokhrel/jira-cli/jira-cli"
    echo "   Other: https://github.com/ankitpokhrel/jira-cli"
    exit 1
fi

# Check JIRA CLI version
jira version
```

#### 2. JIRA Authentication
```bash
# Verify JIRA authentication
jira me
```
**If authentication fails:**
```bash
# Configure JIRA CLI
jira init
# Follow prompts to set:
# - JIRA URL (e.g., https://company.atlassian.net)
# - Username/Email
# - API Token (from https://id.atlassian.com/manage-profile/security/api-tokens)
```

#### 3. X-Ray Integration Check
```bash
# Test X-Ray API access
curl -H "Authorization: Bearer $JIRA_API_TOKEN" \
     "$JIRA_URL/rest/raven/1.0/api/settings"
```
**If X-Ray check fails:**
- Verify X-Ray is installed in your JIRA instance
- Check if you have "Create Test" permissions in X-Ray
- Ensure API token has appropriate scopes

#### 4. Required Environment Variables
```bash
# Check required environment variables
if [[ -z "$JIRA_API_TOKEN" ]]; then
    echo "❌ JIRA_API_TOKEN not set. Export your JIRA API token:"
    echo "   export JIRA_API_TOKEN='your-api-token-here'"
    exit 1
fi

if [[ -z "$JIRA_URL" ]]; then
    echo "❌ JIRA_URL not set. Export your JIRA URL:"
    echo "   export JIRA_URL='https://company.atlassian.net'"
    exit 1
fi
```

#### 5. Ticket and AC Validation
```bash
# Verify ticket exists and is accessible
jira issue view TICKET-123 --plain

# Check if ACs are already available
if [[ -f ".ac-verification/TICKET-123/ac-checklist.md" ]]; then
    echo "✅ Using existing ACs from /collect-ac command"
    AC_SOURCE=".ac-verification/TICKET-123/ac-checklist.md"
elif [[ -n "$(jira issue view TICKET-123 --plain | grep -i 'acceptance criteria')" ]]; then
    echo "⚠️  ACs found in ticket but not extracted yet."
    echo "   Recommend running: /collect-ac TICKET-123 first"
    echo "   Continue anyway? (y/N)"
else
    echo "❌ No acceptance criteria found. Run /collect-ac TICKET-123 first"
    exit 1
fi
```

## Workflow

### Step 1: Load Acceptance Criteria

#### Option A: Use Existing AC Collection (Recommended)
```bash
# Check if ACs were already collected using /collect-ac
if [[ -f ".ac-verification/$TICKET/ac-checklist.md" ]]; then
    echo "✅ Loading ACs from previous /collect-ac run"
    AC_FILE=".ac-verification/$TICKET/ac-checklist.md"
    cp "$AC_FILE" ".xray-tests/$TICKET/acceptance-criteria.md"
else
    echo "❌ No pre-collected ACs found. Please run:"
    echo "   /collect-ac $TICKET"
    echo "   Then run this command again"
    exit 1
fi
```

#### Option B: Quick AC Extraction (Fallback)
If `/collect-ac` wasn't run, perform basic extraction:
```bash
# Simple extraction from JIRA ticket
jira issue view $TICKET --plain > ".xray-tests/$TICKET/ticket-details.txt"

# Look for AC patterns in description
grep -A 20 -i "acceptance criteria\|AC:" ".xray-tests/$TICKET/ticket-details.txt" > \
     ".xray-tests/$TICKET/acceptance-criteria.md"
```

**Note:** For comprehensive AC extraction with multiple sources (Confluence, sub-tasks, etc.), use the dedicated `/collect-ac` command first.

### Step 2: Determine Test Case Format
Based on user preference or AC content analysis:

1. **Format Selection Logic:**
   ```
   If --format=bdd specified:
     Use BDD format
   Else if --format=manual specified:
     Use Manual format
   Else:
     Analyze ACs for Given-When-Then patterns
     If patterns found: Suggest BDD
     Else: Default to Manual
   ```

2. **Ask user for confirmation:**
   ```
   Detected format: BDD (based on Given-When-Then patterns in ACs)
   Proceed with BDD format? (Y/n/manual)
   ```

### Step 3: Generate Test Cases

#### For BDD Format:
```gherkin
Feature: [Ticket Summary]
  As a [user type from ticket]
  I want [main functionality]
  So that [business value]

  Background:
    Given the system is in a valid state
    And the user has appropriate permissions

  Scenario: [AC 1 title]
    Given [preconditions from AC]
    When [action from AC]
    Then [expected result from AC]
    
  Scenario: [AC 2 title]
    Given [preconditions from AC]
    When [action from AC]
    Then [expected result from AC]
```

#### For Manual Format:
```
Test Case: TC-[TICKET]-001 - [AC 1 Summary]
Objective: Verify [what's being tested]
Preconditions: 
  - [Setup requirements]
Test Steps:
  1. [Step 1 action]
  2. [Step 2 action]
Expected Results:
  1. [Expected result 1]
  2. [Expected result 2]
```

### Step 5: Create Tests in X-Ray

1. **Generate X-Ray JSON payload:**
   For BDD tests:
   ```json
   {
     "testType": "Cucumber",
     "projectKey": "PROJECT",
     "summary": "[AC Title] - [Brief Description]",
     "description": "Generated from JIRA ticket acceptance criteria\n\nOriginal AC:\n[Full AC text from ticket]",
     "steps": {
       "cucumber": "[Complete Gherkin Feature with Background, Scenarios, and Examples]"
     },
     "labels": ["automated-generation", "ac-based", "bdd"],
     "components": ["[Component from ticket]"],
     "fixVersions": ["[Version from ticket]"],
     "customFields": {
       "customfield_xxxxx": "[Link back to original story]"
     }
   }
   ```

   **Critical**: The `steps.cucumber` field must contain the **complete Gherkin content** including:
   - Feature description with user story format
   - Background steps (if applicable) 
   - All Scenario steps with Given-When-Then
   - Scenario Outlines with Examples tables
   - Proper Gherkin formatting and indentation

   For Manual tests:
   ```json
   {
     "testType": "Manual",
     "projectKey": "PROJECT", 
     "summary": "[AC Title] - [Brief Description]",
     "description": "Generated from acceptance criteria\n\nOriginal AC:\n[Full AC text from ticket]",
     "steps": [
       {
         "step": "[Detailed action with context]",
         "result": "[Specific expected result with verification criteria]"
       },
       {
         "step": "[Next action step]", 
         "result": "[Expected outcome]"
       }
     ],
     "labels": ["automated-generation", "ac-based", "manual"],
     "components": ["[Component from ticket]"],
     "fixVersions": ["[Version from ticket]"]
   }
   ```

   **Critical**: Manual test steps must be **detailed and actionable**:
   - Each step should be specific enough for any tester to execute
   - Expected results should include specific verification criteria
   - Include setup/teardown steps where needed
   - Reference specific UI elements, API endpoints, or data values

2. **Create tests via X-Ray REST API:**
   ```bash
   # For each generated test case
   curl -X POST \
     -H "Authorization: Bearer $XRAY_TOKEN" \
     -H "Content-Type: application/json" \
     "$JIRA_URL/rest/raven/1.0/api/test" \
     -d "$TEST_JSON"
   ```

3. **Link tests to original ticket:**
   ```bash
   # Link test to story/requirement
   curl -X POST \
     "$JIRA_URL/rest/api/2/issue/$TEST_KEY/remotelink" \
     -d '{"object": {"url": "$JIRA_URL/browse/TICKET-123", "title": "Tests Story"}}'
   ```

### Step 7: Validate Created Tests

1. **Verify X-Ray Test Content:**
   ```bash
   # Check if Gherkin content was properly uploaded
   for test_key in $CREATED_TESTS; do
     echo "Checking $test_key..."
     curl -H "Authorization: Bearer $XRAY_TOKEN" \
          "$JIRA_URL/rest/raven/1.0/api/test/$test_key/step"
   done
   ```

2. **Content Validation Checklist:**
   - ✅ Test summary matches AC title
   - ✅ Description contains original AC text
   - ✅ Steps/Gherkin content is complete and formatted
   - ✅ Labels and components are set correctly
   - ✅ Links to original story are established

3. **Fix Missing Content:**
   If X-Ray tests are missing detailed content:
   ```bash
   # Upload Gherkin content via X-Ray API
   curl -X PUT \
     -H "Authorization: Bearer $XRAY_TOKEN" \
     -H "Content-Type: application/json" \
     "$JIRA_URL/rest/raven/1.0/api/test/$TEST_KEY/step" \
     -d '{"steps": {"cucumber": "'$(cat ac1-cutoff-eligibility.feature)'"}}'
   ```

### Step 8: User Review and Confirmation

1. **Present summary:**
   ```
   ✅ Generated 4 test cases from 4 acceptance criteria
   Format: BDD (Cucumber)
   
   Test Cases Created:
   - TC-PROJ-001: User login validation
   - TC-PROJ-002: Dashboard display verification  
   - TC-PROJ-003: Data export functionality
   - TC-PROJ-004: Error handling validation
   
   Next steps:
   1. Review tests in X-Ray: [X-Ray URL]
   2. Add to Test Set: [Suggested Test Set]
   3. Schedule execution: [Suggested timeline]
   ```

2. **Ask for final actions:**
   ```
   Would you like to:
   1. Create a Test Set with these tests? (Y/n)
   2. Schedule test execution? (Y/n) 
   3. Generate test data templates? (Y/n)
   ```

## Command Options

- `--format=bdd|manual`: Force specific test case format (default: auto-detect from AC content)
- `--dry-run`: Generate test cases but don't create in X-Ray (output to files only)
- `--link-story`: Link generated tests back to the original story (default: true)
- `--environment=ENV`: Set target test environment in test metadata
- `--component=COMP`: Override component detection from ticket

## Output Files

```
.xray-tests/$TICKET/
├── acceptance-criteria.md       # Extracted and cleaned ACs
├── test-cases-bdd.feature      # Generated BDD test cases (if BDD format)
├── test-cases-manual.md        # Generated manual test cases (if Manual format)
├── xray-payload.json           # X-Ray API payloads used
├── creation-log.txt            # Test creation results and X-Ray IDs
├── generation-report.md        # Summary report with links
└── test-plan.json             # Test plan details (if created)
```

## Integration Points

1. **AC Collection Integration:**
   - **Prerequisite**: Use `/collect-ac TICKET-123` first to extract and verify ACs
   - Reads from `.ac-verification/$TICKET/ac-checklist.md` when available
   - Falls back to basic JIRA description parsing if needed

2. **JIRA Integration:**
   - Fetches ticket metadata via JIRA CLI
   - Links tests back to original story
   - Uses ticket components and fix versions

3. **X-Ray Integration:**
   - Creates tests via X-Ray REST API
   - Supports both BDD (Cucumber) and Manual test types
   - Proper Gherkin content upload to test steps

## Error Handling

1. **Ticket Access Issues:**
   - Verify ticket exists and user has access
   - Provide clear error messages with resolution steps

2. **X-Ray API Failures:**
   - Retry logic with exponential backoff
   - Fall back to file output if API unavailable
   - Clear error messages for permission issues

3. **AC Extraction Failures:**
   - Fall back to manual AC entry
   - Provide guidance on AC format requirements

4. **Missing Gherkin Content in X-Ray:**
   - **Issue**: X-Ray test cases show only summary/description but no detailed scenarios
   - **Cause**: API payload missing `steps.cucumber` content or using wrong endpoint
   - **Solution**: Ensure complete Gherkin content is included in `steps.cucumber` field
   - **Verification**: Check X-Ray test case "Steps" tab for Gherkin scenarios
   - **Manual Fix**: Upload .feature files via X-Ray UI if API fails

5. **Test Case Content Issues:**
   - **Empty/Minimal Steps**: Verify AC parsing extracted meaningful content
   - **Format Issues**: Validate Gherkin syntax before API submission  
   - **Missing Details**: Ensure original ACs contain sufficient detail for test generation

## Example Usage

```bash
# Prerequisites: Extract ACs first
/collect-ac PROJ-123

# Basic usage - auto-detect format from AC content
/generate-xray-tests PROJ-123

# Force BDD format
/generate-xray-tests PROJ-123 --format=bdd

# Dry run to review before creation
/generate-xray-tests PROJ-123 --dry-run

# Create with specific environment
/generate-xray-tests PROJ-123 --environment=staging
```

## Success Metrics

- Number of test cases successfully created in X-Ray
- Time saved vs manual test case creation
- Quality of generated test coverage
- Successful linking between stories and tests