---
name: /generate-xray-tests
argument-hint: 'TICKET-123 [--format=bdd|manual] [--dry-run]'
description: Generate X-Ray test cases from JIRA ticket acceptance criteria
mode: single-agent
dependencies:
  - jira CLI (required)
  - X-Ray for JIRA (required)
---

# Generate X-Ray Test Cases from JIRA Tickets

## Purpose
This command generates X-Ray test cases directly from a JIRA ticket's acceptance criteria. It extracts ACs from the ticket, converts them into test case format (BDD or Manual), and creates the test cases in X-Ray with proper linking back to the original ticket.

**Single Purpose:** Create X-Ray test cases only - no reports, no test executions, no test plans.

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

# Check if ticket has ACs in description or custom fields
TICKET_CONTENT=$(jira issue view TICKET-123 --plain)
if [[ -n "$(echo "$TICKET_CONTENT" | grep -i 'acceptance criteria\|AC:')" ]]; then
    echo "✅ Acceptance criteria found in ticket"
else
    echo "⚠️  No clear acceptance criteria found in ticket."
    echo "   Will attempt to generate test cases from description and requirements"
fi
```

## Workflow

### Step 1: Extract Acceptance Criteria from JIRA Ticket

1. **Fetch complete ticket details:**
   ```bash
   # Get ticket content including description, custom fields, and metadata
   TICKET_DATA=$(jira issue view $TICKET --plain)
   TICKET_SUMMARY=$(echo "$TICKET_DATA" | grep "SUMMARY" | cut -d: -f2-)
   TICKET_DESCRIPTION=$(echo "$TICKET_DATA" | sed -n '/DESCRIPTION/,/ASSIGNEE/p')
   ```

2. **Extract Acceptance Criteria in memory:**
   ```bash
   # Look for AC patterns in multiple locations:
   
   # 1. Custom field "Acceptance Criteria"
   AC_CUSTOM_FIELD=$(echo "$TICKET_DATA" | grep -A 50 "Acceptance Criteria:")
   
   # 2. AC section in description
   AC_IN_DESCRIPTION=$(echo "$TICKET_DESCRIPTION" | grep -A 20 -i "acceptance criteria\|^AC[0-9]\|^- AC")
   
   # 3. Structured requirements in description
   REQUIREMENTS=$(echo "$TICKET_DESCRIPTION" | grep -A 20 -i "requirements\|user story\|as a.*i want")
   
   # Combine extracted ACs into array
   readarray -t ACS_ARRAY <<< "$(echo -e "$AC_CUSTOM_FIELD\n$AC_IN_DESCRIPTION\n$REQUIREMENTS" | grep -v '^$')"
   AC_COUNT=${#ACS_ARRAY[@]}
   echo "Found $AC_COUNT acceptance criteria"
   ```

### Step 2: Determine Test Case Format and Generate Content

1. **Format Selection Logic:**
   ```bash
   if [[ "$FORMAT" == "bdd" ]] || [[ -z "$FORMAT" && -n "$(grep -i 'given.*when.*then' ".xray-tests/$TICKET/acceptance-criteria.md")" ]]; then
       echo "Using BDD (Gherkin) format"
       TEST_FORMAT="bdd"
   elif [[ "$FORMAT" == "manual" ]] || [[ -z "$FORMAT" ]]; then
       echo "Using Manual test format"
       TEST_FORMAT="manual"
   fi
   ```

2. **Generate Test Cases for Each AC:**

   **For BDD Format:**
   ```gherkin
   Feature: [TICKET] - [Summary]
     As a [user type from ticket]
     I want [main functionality]
     So that [business value]

   Background:
     Given the system is in a valid state
     And the user has appropriate permissions

   Scenario: [AC1 converted to scenario name]
     Given [preconditions from AC]
     When [action from AC]  
     Then [expected result from AC]
     
   Scenario: [AC2 converted to scenario name]
     Given [preconditions from AC]
     When [action from AC]
     Then [expected result from AC]
   ```

   **For Manual Format:**
   ```
   Test Case: [TICKET]-TC-001 - [AC1 Summary]
   
   Objective: Verify [what's being tested from AC]
   
   Preconditions:
   - [Setup requirements from ticket context]
   
   Test Steps:
   1. [Convert AC into actionable step]
   2. [Next logical test action]
   3. [Verification step]
   
   Expected Results:
   1. [Expected outcome from AC]
   2. [System behavior verification]
   3. [Final validation]
   ```

### Step 3: Generate and Create X-Ray Test Cases

Generate and create test cases directly in X-Ray for each acceptance criteria:

```bash
CREATED_TESTS=()

# For each identified AC, generate test case content and create in X-Ray
for i in "${!ACS_ARRAY[@]}"; do
    AC_TEXT="${ACS_ARRAY[$i]}"
    AC_TITLE=$(echo "$AC_TEXT" | head -1 | sed 's/^AC[0-9]*[:.]\s*//')
    TEST_TITLE="${TICKET}-TC-$(printf "%03d" $((i+1))) - ${AC_TITLE}"
    
    if [[ "$TEST_FORMAT" == "bdd" ]]; then
        # Generate BDD content in memory
        GHERKIN_CONTENT=$(cat <<EOF
Feature: $TEST_TITLE

  Background:
    Given the system is in a valid state
    And the user has appropriate permissions

  Scenario: $AC_TITLE
    Given $(echo "$AC_TEXT" | grep -i 'given\|precondition' | head -1 | sed 's/.*given\|.*precondition//i' | xargs)
    When $(echo "$AC_TEXT" | grep -i 'when\|user\|action' | head -1 | sed 's/.*when\|.*user\|.*action//i' | xargs)
    Then $(echo "$AC_TEXT" | grep -i 'then\|should\|expected' | head -1 | sed 's/.*then\|.*should\|.*expected//i' | xargs)
EOF
)
        
        # Create BDD test case in X-Ray
        JSON_PAYLOAD=$(jq -n \
            --arg projectKey "${TICKET%%-*}" \
            --arg summary "$TEST_TITLE" \
            --arg description "Generated from JIRA ticket $TICKET acceptance criteria: $AC_TEXT" \
            --arg cucumber "$GHERKIN_CONTENT" \
            --arg ticket "$TICKET" \
            '{
                testType: "Cucumber",
                projectKey: $projectKey,
                summary: $summary,
                description: $description,
                steps: {cucumber: $cucumber},
                labels: ["automated-generation", "ac-based", "bdd"],
                issueLinks: [{type: "Test", inwardIssue: $ticket}]
            }')
    else
        # Generate Manual test case content in memory
        TEST_STEPS='[
            {"step": "Navigate to the feature area mentioned in AC", "result": "Feature area is accessible"},
            {"step": "Execute the action described in: '"$AC_TEXT"'", "result": "Action completes successfully"},
            {"step": "Verify the expected outcome from AC", "result": "Expected behavior is observed"}
        ]'
        
        # Create Manual test case in X-Ray  
        JSON_PAYLOAD=$(jq -n \
            --arg projectKey "${TICKET%%-*}" \
            --arg summary "$TEST_TITLE" \
            --arg description "Generated from JIRA ticket $TICKET acceptance criteria: $AC_TEXT" \
            --argjson steps "$TEST_STEPS" \
            --arg ticket "$TICKET" \
            '{
                testType: "Manual",
                projectKey: $projectKey,
                summary: $summary,
                description: $description,
                steps: $steps,
                labels: ["automated-generation", "ac-based", "manual"],
                issueLinks: [{type: "Test", inwardIssue: $ticket}]
            }')
    fi
    
    # Create test case in X-Ray
    echo "Creating test case: $TEST_TITLE..."
    RESPONSE=$(curl -s -X POST \
        -H "Authorization: Bearer $JIRA_API_TOKEN" \
        -H "Content-Type: application/json" \
        "$JIRA_URL/rest/raven/1.0/api/test" \
        -d "$JSON_PAYLOAD")
    
    TEST_KEY=$(echo "$RESPONSE" | jq -r '.key // empty')
    if [[ -n "$TEST_KEY" && "$TEST_KEY" != "null" ]]; then
        echo "✅ Created X-Ray test: $TEST_KEY"
        CREATED_TESTS+=("$TEST_KEY")
        
        # Link test to original JIRA ticket
        curl -s -X POST \
            -H "Authorization: Bearer $JIRA_API_TOKEN" \
            -H "Content-Type: application/json" \
            "$JIRA_URL/rest/api/2/issueLink" \
            -d "{
                \"type\": {\"name\": \"Test\"},
                \"inwardIssue\": {\"key\": \"$TICKET\"},
                \"outwardIssue\": {\"key\": \"$TEST_KEY\"}
            }" > /dev/null
        echo "🔗 Linked $TEST_KEY to $TICKET"
    else
        echo "❌ Failed to create test case for AC $((i+1))"
        echo "Response: $RESPONSE"
    fi
done
```

### Step 4: Display Results

```bash
# Display completion summary
echo ""
echo "✅ Generated ${#CREATED_TESTS[@]} X-Ray test cases from $AC_COUNT acceptance criteria"
echo "✅ All tests linked to $TICKET"  
echo "✅ Test format: $TEST_FORMAT"
echo ""
echo "Created tests:"
for test_key in "${CREATED_TESTS[@]}"; do
    echo "  - $test_key"
done
echo ""
echo "View tests in X-Ray: $JIRA_URL/browse/$TICKET (see linked tests)"
```

### Step 5: Summary and Completion

1. **Display completion summary:**
   ```bash
   echo "✅ Generated ${#CREATED_TESTS[@]} X-Ray test cases"
   echo "✅ All tests linked to $TICKET"
   echo "✅ Test format: $TEST_FORMAT"
   echo ""
   echo "Created tests:"
   for test_key in "${CREATED_TESTS[@]}"; do
     echo "  - $test_key"
   done
   ```

**Note:** No local files are created. All test cases are generated and created directly in X-Ray.

## Command Options

- `--format=bdd|manual`: Force specific test case format (default: auto-detect from AC content)
- `--dry-run`: Generate test cases but don't create in X-Ray (output to files only)
- `--environment=ENV`: Set target test environment in test metadata
- `--component=COMP`: Override component detection from ticket

## Output Files

**No local files are created.** The command works entirely in memory and creates test cases directly in X-Ray.

The only output is the X-Ray test cases created in your JIRA instance, properly linked to the original ticket.

## Integration Points

1. **JIRA Integration:**
   - **Input:** Extracts ACs and requirements directly from JIRA ticket
   - **Linking:** Creates issue links between generated X-Ray tests and original ticket
   - **Metadata:** Uses ticket project key, components, and fix versions

2. **X-Ray Integration:**
   - **Test Creation:** Creates tests via X-Ray REST API with complete content
   - **Content Upload:** Includes full Gherkin scenarios or detailed manual steps
   - **Linking:** Establishes "Test" relationship between X-Ray tests and JIRA story

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

5. **AC Extraction Failures:**
   - Fall back to manual AC entry if no patterns found
   - Provide guidance on AC format requirements

6. **Missing Content Issues:**
   - **Issue**: No acceptance criteria found in ticket
   - **Cause**: Ticket lacks structured AC content
   - **Solution**: Provide guidance on where to add ACs in ticket
   - **Manual Fix**: User adds ACs to ticket and re-runs command

## Example Usage

```bash
# Basic usage - extract ACs from ticket and generate X-Ray tests
/generate-xray-tests PROJ-123

# Force BDD format
/generate-xray-tests PROJ-123 --format=bdd

# Force Manual format  
/generate-xray-tests PROJ-123 --format=manual

# Dry run to review generated content before creating in X-Ray
/generate-xray-tests PROJ-123 --dry-run

# Create tests with specific environment metadata
/generate-xray-tests PROJ-123 --environment=staging
```

## Success Metrics

- Number of X-Ray test cases successfully created
- Successful linking between X-Ray tests and original JIRA ticket
- Quality of generated test case content (scenarios/steps)