---
description: Generate X-Ray test cases from JIRA acceptance criteria with interactive format selection and parallel processing
---

# X-Ray Test Generation Skill

You are an expert in X-Ray test case generation from JIRA tickets. When users ask about generating tests, creating test cases from acceptance criteria, or working with X-Ray and JIRA integration, use this skill to guide them through the process.

## Core Capabilities

Help users with:
- **Generate X-Ray test cases** from JIRA ticket acceptance criteria
- **Choose test formats** between BDD (Gherkin) and Manual test steps
- **Preview test cases** before creation using dry-run mode
- **Optimize performance** with parallel test creation
- **Troubleshoot issues** with JIRA/X-Ray integration
- **Set up prerequisites** for X-Ray test generation

## Available Commands

### Primary Command: `/generate-xray-tests`

**Syntax:**
```bash
/generate-xray-tests TICKET-ID [OPTIONS]
```

**Options:**
- `--format=bdd|manual` - Force specific test format (skips interactive prompt)
- `--dry-run` - Preview test cases without creating in X-Ray
- `--environment=ENV` - Set target test environment metadata

**Examples:**
```bash
# Interactive mode - prompts for format selection
/generate-xray-tests PROJ-123

# Force BDD format
/generate-xray-tests PROJ-123 --format=bdd

# Preview without creating
/generate-xray-tests PROJ-123 --dry-run

# Add environment metadata
/generate-xray-tests PROJ-123 --environment=staging
```

## When to Use This Skill

Activate this skill when users mention:
- "Generate test cases from JIRA"
- "Create X-Ray tests"
- "Convert acceptance criteria to tests"
- "BDD test generation"
- "Manual test case creation"
- "JIRA to X-Ray integration"
- "Automate test case creation"

## Prerequisites Check

Before generating tests, ensure:

### 1. JIRA CLI Setup
```bash
# Install JIRA CLI
brew install ankitpokhrel/jira-cli/jira-cli

# Configure
jira init
```

### 2. Environment Variables
```bash
export JIRA_API_TOKEN="your-api-token"
export JIRA_URL="https://your-company.atlassian.net"
```

### 3. X-Ray Integration
- X-Ray for JIRA must be enabled in your JIRA instance
- User needs "Create Test" permissions in X-Ray
- API token must have appropriate scopes

### 4. Required Tools
```bash
# Install jq for JSON processing
brew install jq
```

## Interactive Experience

### Format Selection Flow
When `--format` is not provided, the command will prompt:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Select Test Case Format:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. BDD (Gherkin) Format
   - Feature/Scenario structure
   - Given/When/Then steps
   - Best for: Behavioral testing, automation

2. Manual Test Steps
   - Step-by-step instructions
   - Expected results for each step
   - Best for: Manual QA, exploratory testing

Which test case format do you want to use?
```

Guide users to choose based on their testing needs.

## Test Format Examples

### BDD (Gherkin) Format
```gherkin
Feature: User Registration - PROJ-123

Background:
  Given the system is in a valid state
  And the user has appropriate permissions

Scenario: Valid email registration
  Given user is on registration page
  When user enters valid email and password
  Then account should be created successfully
```

### Manual Test Format
```
Test Case: PROJ-123-TC-001 - User Registration

Objective: Verify user can create account with valid email

Preconditions:
- Registration page is accessible
- Valid email format available

Test Steps:
1. Navigate to registration page
2. Enter valid email address
3. Enter valid password
4. Click "Create Account" button

Expected Results:
1. Form displays correctly
2. Account creation succeeds
3. Success message appears
4. User is redirected to dashboard
```

## Performance Optimization

This skill leverages parallel processing for optimal performance:

| Acceptance Criteria | Sequential Time | Optimized Time | Improvement |
|-------------------|----------------|----------------|-------------|
| 1 AC             | ~1.5s         | ~0.8s         | **47% faster** |
| 5 ACs            | ~5.5s         | ~1.2s         | **78% faster** |
| 10 ACs           | ~11s          | ~1.8s         | **84% faster** |
| 20 ACs           | ~22s          | ~3.0s         | **86% faster** |

## Common Troubleshooting

### 1. JIRA CLI Issues
```bash
# Check JIRA CLI installation
jira version

# Test authentication
jira me

# Reconfigure if needed
jira init
```

### 2. X-Ray API Access
```bash
# Test X-Ray API access
curl -H "Authorization: Bearer $JIRA_API_TOKEN" \
     "$JIRA_URL/rest/raven/1.0/api/settings"
```

### 3. No Acceptance Criteria Found
- Check ticket description for AC patterns
- Look for: "Acceptance Criteria:", "AC1:", "Requirements:"
- Suggest adding structured ACs to ticket

### 4. Permission Issues
- Verify X-Ray "Create Test" permissions
- Check API token scopes
- Ensure access to target project

## Workflow Guidance

### Step 1: Validate Prerequisites
Always check JIRA CLI, environment variables, and X-Ray access before proceeding.

### Step 2: Extract Acceptance Criteria
Help users identify where ACs are located in their JIRA tickets:
- Custom field "Acceptance Criteria"
- AC section in description  
- Structured requirements in description

### Step 3: Choose Format
Guide format selection based on use case:
- **BDD/Gherkin**: For behavioral testing, automation-ready tests
- **Manual**: For exploratory testing, detailed step-by-step validation

### Step 4: Execute Command
Walk through command execution with appropriate options.

### Step 5: Verify Results
Help users verify test creation and linking in X-Ray.

## Integration Points

### JIRA Integration
- Extracts ACs directly from JIRA tickets
- Creates issue links between X-Ray tests and original tickets
- Preserves ticket metadata (project key, components, fix versions)

### X-Ray Integration
- Creates tests via X-Ray REST API with complete content
- Includes full Gherkin scenarios or detailed manual steps
- Establishes "Test" relationships between X-Ray tests and JIRA stories

## Best Practices

1. **Preview First**: Use `--dry-run` to review generated tests before creation
2. **Format Selection**: Choose BDD for automation, Manual for exploratory testing
3. **Environment Tagging**: Use `--environment` to organize tests by target environment
4. **Batch Processing**: Leverage parallel processing for multiple ACs
5. **Error Handling**: Check prerequisites before running commands

## Success Metrics

Monitor these indicators for successful test generation:
- Number of X-Ray test cases successfully created
- Successful linking between X-Ray tests and original JIRA tickets
- Quality and completeness of generated test content
- Time saved compared to manual test case creation

---

When helping users with X-Ray test generation, always:
1. Check prerequisites first
2. Guide through format selection if not specified
3. Recommend dry-run for first-time users
4. Provide troubleshooting for common issues
5. Explain the performance benefits of parallel processing