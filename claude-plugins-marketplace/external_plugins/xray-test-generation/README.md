# X-Ray Test Generation Plugin

> Automated X-Ray test case generation from JIRA ticket acceptance criteria for HelloFresh Claude Code

## 🎯 What It Does

Generate X-Ray test cases directly from JIRA tickets with interactive format selection, parallel processing, and automatic ticket linking.

### ✨ Key Features

- **🔗 JIRA Integration** - Extract acceptance criteria directly from JIRA tickets
- **⚡ Interactive Format Selection** - Choose between BDD (Gherkin) or Manual test formats
- **🚀 Parallel Processing** - Create multiple test cases simultaneously (6-10x faster)
- **🔗 Automatic Linking** - Link generated tests back to original JIRA tickets
- **👀 Dry-Run Mode** - Preview test cases before creating them in X-Ray
- **🏷️ Environment Support** - Tag tests with specific environment metadata

## 📦 Installation

Add the HelloFresh plugin marketplace to Claude Code:

```bash
claude plugin marketplace add "hellofresh/claude-plugins-marketplace"
```

Then install the X-Ray Test Generation plugin:

```bash
claude plugin install xray-test-generation
```

## 🔧 Prerequisites

### Required Tools

- **JIRA CLI** - For JIRA integration
  ```bash
  # macOS
  brew install ankitpokhrel/jira-cli/jira-cli
  
  # Configure
  jira init
  ```

- **X-Ray for JIRA** - Must be enabled in your JIRA instance
- **jq** - For JSON processing
  ```bash
  brew install jq
  ```

### Environment Variables

```bash
export JIRA_API_TOKEN="your-api-token"
export JIRA_URL="https://your-company.atlassian.net"
```

## 🚀 Usage

### Basic Usage

```bash
# Interactive mode - prompts for format selection
/generate-xray-tests PROJ-123

# Force BDD format (skip prompt)
/generate-xray-tests PROJ-123 --format=bdd

# Force Manual format (skip prompt)  
/generate-xray-tests PROJ-123 --format=manual
```

### Advanced Options

```bash
# Preview without creating (dry-run)
/generate-xray-tests PROJ-123 --dry-run

# Add environment metadata
/generate-xray-tests PROJ-123 --environment=staging

# Combine options
/generate-xray-tests PROJ-123 --format=bdd --environment=production
```

## 📋 Test Formats

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

## ⚡ Performance

The plugin uses optimized parallel processing:

| Acceptance Criteria | Sequential Time | Optimized Time | Improvement |
|-------------------|----------------|----------------|-------------|
| 1 AC             | ~1.5s         | ~0.8s         | **47% faster** |
| 5 ACs            | ~5.5s         | ~1.2s         | **78% faster** |
| 10 ACs           | ~11s          | ~1.8s         | **84% faster** |
| 20 ACs           | ~22s          | ~3.0s         | **86% faster** |

## 🔧 Command Options

- `--format=bdd|manual` - Force specific test format (skips interactive prompt)
- `--dry-run` - Preview test cases without creating in X-Ray
- `--environment=ENV` - Set target test environment metadata
- `--component=COMP` - Override component detection from ticket

## 📖 Interactive Mode

Without the `--format` flag, the command prompts you to choose:

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

## 🎯 Integration Points

1. **JIRA Integration:**
   - Extracts ACs and requirements directly from JIRA tickets
   - Creates issue links between generated X-Ray tests and original tickets
   - Uses ticket metadata (project key, components, fix versions)

2. **X-Ray Integration:**
   - Creates tests via X-Ray REST API with complete content
   - Includes full Gherkin scenarios or detailed manual steps
   - Establishes "Test" relationships between X-Ray tests and JIRA stories

## 🐛 Troubleshooting

### Common Issues

**❌ JIRA CLI not found**
```bash
# Install JIRA CLI
brew install ankitpokhrel/jira-cli/jira-cli
jira init
```

**❌ Authentication failed**
```bash
# Check JIRA authentication
jira me

# Reconfigure if needed
jira init
```

**❌ X-Ray API access denied**
- Verify X-Ray is installed in your JIRA instance
- Check if you have "Create Test" permissions in X-Ray
- Ensure API token has appropriate scopes

**❌ No acceptance criteria found**
- Add acceptance criteria to the ticket description
- Use supported AC patterns: "AC1:", "Acceptance Criteria:", "Requirements:"

## 📝 Version History

### v2.2.0 (Current)
- ✨ Interactive format selection (BDD vs Manual)
- ⚡ Parallel processing for 6-10x performance improvement
- 🔗 Automatic ticket linking and metadata preservation
- 👀 Dry-run mode for preview before creation
- 🏷️ Environment-specific test tagging
- 🛠️ Comprehensive error handling

## 🤝 Contributing

This plugin is part of the HelloFresh Claude Plugin marketplace. To contribute:

1. Fork the marketplace repository
2. Make your changes in `external_plugins/xray-test-generation/`
3. Update the version in `plugin.json`
4. Submit a PR

## 📞 Support

For issues and feature requests:
- **Slack**: `#squad-quality-engineering`
- **Email**: qa-team@hellofresh.com

## 📄 License

MIT License - See original repository for full details.