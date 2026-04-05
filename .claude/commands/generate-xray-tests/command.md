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
   jira issue view $TICKET --plain > ".xray-tests/$TICKET/ticket-details.txt"
   
   # Extract ticket metadata
   TICKET_SUMMARY=$(jira issue view $TICKET --plain | grep "SUMMARY" | cut -d: -f2-)
   TICKET_DESCRIPTION=$(jira issue view $TICKET --plain | sed -n '/DESCRIPTION/,/ASSIGNEE/p')
   ```

2. **Extract Acceptance Criteria:**
   ```bash
   # Look for AC patterns in multiple locations:
   
   # 1. Custom field "Acceptance Criteria"
   AC_CUSTOM_FIELD=$(jira issue view $TICKET --plain | grep -A 50 "Acceptance Criteria:")
   
   # 2. AC section in description
   AC_IN_DESCRIPTION=$(echo "$TICKET_DESCRIPTION" | grep -A 20 -i "acceptance criteria\|^AC[0-9]\|^- AC")
   
   # 3. Structured requirements in description
   REQUIREMENTS=$(echo "$TICKET_DESCRIPTION" | grep -A 20 -i "requirements\|user story\|as a.*i want")
   
   # Combine and clean up
   echo "# Acceptance Criteria for $TICKET" > ".xray-tests/$TICKET/acceptance-criteria.md"
   echo "## Source: $TICKET_SUMMARY" >> ".xray-tests/$TICKET/acceptance-criteria.md"
   echo "" >> ".xray-tests/$TICKET/acceptance-criteria.md"
   
   # Add extracted ACs
   [[ -n "$AC_CUSTOM_FIELD" ]] && echo "$AC_CUSTOM_FIELD" >> ".xray-tests/$TICKET/acceptance-criteria.md"
   [[ -n "$AC_IN_DESCRIPTION" ]] && echo "$AC_IN_DESCRIPTION" >> ".xray-tests/$TICKET/acceptance-criteria.md"
   [[ -n "$REQUIREMENTS" ]] && echo "$REQUIREMENTS" >> ".xray-tests/$TICKET/acceptance-criteria.md"
   ```

3. **Parse and structure ACs:**
   ```bash
   # Parse individual ACs from the extracted content
   # Look for patterns like:
   # - AC1: ...
   # - Given... When... Then...
   # - As a user I want...
   # - The system should...
   
   # Number each AC for reference
   AC_COUNT=$(grep -c -E "^(AC[0-9]|Given|As a|The system)" ".xray-tests/$TICKET/acceptance-criteria.md")
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

### Step 3: Generate Test Cases

Generate individual test case files for each acceptance criteria:

```bash
# For each identified AC, create a separate test case
for i in $(seq 1 $AC_COUNT); do
    AC_TEXT=$(sed -n "${i}p" ".xray-tests/$TICKET/parsed-acs.txt")
    
    if [[ "$TEST_FORMAT" == "bdd" ]]; then
        # Create BDD feature file for this AC
        cat > ".xray-tests/$TICKET/tc-$(printf "%03d" $i)-${AC_TITLE}.feature" <<EOF
Feature: $TICKET-TC-$(printf "%03d" $i) - $AC_TITLE

  Background:
    Given the system is in a valid state
    And the user has appropriate permissions

  Scenario: $SCENARIO_NAME
    Given $GIVEN_CONDITION
    When $WHEN_ACTION
    Then $THEN_RESULT
EOF
    else
        # Create Manual test case
        cat > ".xray-tests/$TICKET/tc-$(printf "%03d" $i)-${AC_TITLE}.md" <<EOF
# Test Case: $TICKET-TC-$(printf "%03d" $i)

## Summary
$AC_TITLE

## Objective  
Verify $AC_OBJECTIVE

## Preconditions
- User has appropriate permissions
- System is in valid state
- $SPECIFIC_PRECONDITIONS

## Test Steps
1. $TEST_STEP_1
2. $TEST_STEP_2
3. $TEST_STEP_3

## Expected Results
1. $EXPECTED_RESULT_1
2. $EXPECTED_RESULT_2
3. $EXPECTED_RESULT_3
EOF
    fi
done
```

### Step 4: Create Test Cases in X-Ray

1. **Generate X-Ray JSON payload for each test case:**
   ```bash
   for test_file in .xray-tests/$TICKET/tc-*.{feature,md}; do
       TEST_TITLE=$(basename "$test_file" | sed 's/tc-[0-9]*-//' | sed 's/\.(feature|md)$//')
       
       if [[ "$test_file" == *.feature ]]; then
           # BDD Test Case
           GHERKIN_CONTENT=$(cat "$test_file")
           JSON_PAYLOAD=$(cat <<EOF
   {
     "testType": "Cucumber",
     "projectKey": "${TICKET%%-*}",
     "summary": "$TEST_TITLE",
     "description": "Generated from JIRA ticket $TICKET acceptance criteria",
     "steps": {
       "cucumber": "$GHERKIN_CONTENT"
     },
     "labels": ["automated-generation", "ac-based", "bdd"],
     "issueLinks": [
       {
         "type": "Test",
         "inwardIssue": "$TICKET"
       }
     ]
   }
   EOF
   )
       else
           # Manual Test Case  
           TEST_STEPS=$(grep -A 20 "## Test Steps" "$test_file" | tail -n +2)
           JSON_PAYLOAD=$(cat <<EOF
   {
     "testType": "Manual", 
     "projectKey": "${TICKET%%-*}",
     "summary": "$TEST_TITLE",
     "description": "Generated from JIRA ticket $TICKET acceptance criteria",
     "steps": [
       $(echo "$TEST_STEPS" | sed 's/^[0-9]*\. /{"step": "/' | sed 's/$/", "result": "Verify step completed successfully"},/')
     ],
     "labels": ["automated-generation", "ac-based", "manual"],
     "issueLinks": [
       {
         "type": "Test", 
         "inwardIssue": "$TICKET"
       }
     ]
   }
   EOF
   )
       fi
       
       echo "$JSON_PAYLOAD" > ".xray-tests/$TICKET/payload-$TEST_TITLE.json"
   done
   ```

2. **Create tests via X-Ray REST API:**
   ```bash
   for payload_file in .xray-tests/$TICKET/payload-*.json; do
       echo "Creating test case from $payload_file..."
       
       RESPONSE=$(curl -X POST \
         -H "Authorization: Bearer $JIRA_API_TOKEN" \
         -H "Content-Type: application/json" \
         "$JIRA_URL/rest/raven/1.0/api/test" \
         -d "@$payload_file")
       
       TEST_KEY=$(echo "$RESPONSE" | jq -r '.key')
       echo "✅ Created X-Ray test: $TEST_KEY"
       echo "$TEST_KEY" >> ".xray-tests/$TICKET/created-tests.txt"
   done
   ```

3. **Link tests to original JIRA ticket:**
   ```bash
   while read TEST_KEY; do
       curl -X POST \
         -H "Authorization: Bearer $JIRA_API_TOKEN" \
         -H "Content-Type: application/json" \
         "$JIRA_URL/rest/api/2/issueLink" \
         -d "{
           \"type\": {\"name\": \"Test\"},
           \"inwardIssue\": {\"key\": \"$TICKET\"},
           \"outwardIssue\": {\"key\": \"$TEST_KEY\"}
         }"
       echo "🔗 Linked $TEST_KEY to $TICKET"
   done < ".xray-tests/$TICKET/created-tests.txt"
   ```

### Step 5: Summary and Completion

1. **Display completion summary:**
   ```bash
   echo "✅ Generated $(wc -l < ".xray-tests/$TICKET/created-tests.txt") X-Ray test cases"
   echo "✅ All tests linked to $TICKET"
   echo "✅ Test format: $TEST_FORMAT"
   echo ""
   echo "Created tests:"
   while read test_key; do
     echo "  - $test_key"
   done < ".xray-tests/$TICKET/created-tests.txt"
   ```

2. **Clean up temporary files:**
   ```bash
   # Keep only essential files, remove processing artifacts
   rm -f ".xray-tests/$TICKET/ticket-details.txt" 
   rm -f ".xray-tests/$TICKET/parsed-acs.txt"
   rm -f ".xray-tests/$TICKET/payload-*.json"
   ```

## Command Options

- `--format=bdd|manual`: Force specific test case format (default: auto-detect from AC content)
- `--dry-run`: Generate test cases but don't create in X-Ray (output to files only)
- `--environment=ENV`: Set target test environment in test metadata
- `--component=COMP`: Override component detection from ticket

## Output Files

```
.xray-tests/$TICKET/
├── acceptance-criteria.md       # Extracted ACs from ticket
├── test-cases-bdd.feature      # Generated BDD test cases (if BDD format)
├── test-cases-manual.md        # Generated manual test cases (if Manual format)
└── created-tests.txt           # List of created X-Ray test keys
```

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