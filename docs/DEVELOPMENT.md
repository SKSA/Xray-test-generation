# Development Guide

This guide explains how to develop and test changes to the X-Ray Test Generator.

## Quick Start

### Prerequisites

1. **JIRA CLI** - Required for testing
   ```bash
   # macOS
   brew install ankitpokhrel/jira-cli/jira-cli
   
   # Other platforms
   # See: https://github.com/ankitpokhrel/jira-cli
   ```

2. **JIRA Authentication**
   ```bash
   jira init
   # Follow prompts to configure
   ```

3. **Environment Variables**
   ```bash
   export JIRA_API_TOKEN="your-api-token"
   export JIRA_URL="https://your-company.atlassian.net"
   ```

4. **Claude Code** - AI coding assistant with command support

## Testing Local Changes

### Method 1: Direct Copy (Fastest)

```bash
# Copy to your test project
cp -r .claude ~/test-project/

cd ~/test-project

# Test in Claude Code
/generate-xray-tests PROJ-123
```

### Method 2: Symlink (Best for Active Development)

```bash
cd ~/test-project

# Create symlink to your development copy
ln -s ~/Xray-test-generation/.claude .claude

# Changes are immediately available
# Edit files in ~/Xray-test-generation/
# Test in ~/test-project/
```

### Method 3: Git Clone

```bash
cd ~/test-project

git clone https://github.com/SKSA/Xray-test-generation .claude-xray
ln -s .claude-xray/.claude .claude

# Update when needed
cd .claude-xray && git pull
```

## Testing Workflow

### Basic Test

1. **Pick a test ticket** with acceptance criteria
   ```bash
   jira issue view PROJ-123
   ```

2. **Run the command**
   ```bash
   /generate-xray-tests PROJ-123
   ```

3. **Verify results**
   - Check Claude's output for created test keys
   - Visit JIRA to see linked tests
   - Verify test content matches acceptance criteria

### Comprehensive Test Checklist

Before submitting a PR, test:

- [ ] **Interactive mode** - No --format flag, select BDD
- [ ] **Interactive mode** - No --format flag, select Manual
- [ ] **BDD format** - `--format=bdd`
- [ ] **Manual format** - `--format=manual`
- [ ] **Dry run** - `--dry-run` (no tests created)
- [ ] **Environment tag** - `--environment=staging`
- [ ] **Multiple ACs** - Ticket with 5+ acceptance criteria
- [ ] **Single AC** - Ticket with 1 acceptance criterion
- [ ] **No ACs** - Ticket without explicit ACs (should handle gracefully)
- [ ] **Error handling** - Invalid ticket ID
- [ ] **Error handling** - Network issues (simulate with wrong JIRA_URL)

## Development Workflow

### Making Changes

1. **Create a feature branch**
   ```bash
   git checkout -b feat/your-feature-name
   ```

2. **Edit the command**
   ```bash
   # Main command implementation
   vi .claude/commands/generate-xray-tests/command.md
   ```

3. **Test your changes**
   ```bash
   # Use Method 2 (symlink) for rapid iteration
   /generate-xray-tests TEST-TICKET
   ```

4. **Update documentation**
   ```bash
   # Update if needed
   vi README.md
   vi docs/GENERATE-XRAY-TESTS-GUIDE.md
   vi CHANGELOG.md
   ```

5. **Run quality checks**
   ```bash
   npm run lint
   npm test
   ```

6. **Commit and push**
   ```bash
   git add .
   git commit -m "feat: Add new feature"
   git push origin feat/your-feature-name
   ```

### Validation Checklist

Before submitting a PR:

- [ ] **Tested with real JIRA ticket**
- [ ] **Both formats work** (BDD and Manual)
- [ ] **Dry-run mode works**
- [ ] **X-Ray test creation successful**
- [ ] **Tests properly linked to ticket**
- [ ] **Documentation updated**
- [ ] **CHANGELOG.md updated**
- [ ] **Lint checks pass** (`npm run lint`)
- [ ] **Tests pass** (`npm test`)
- [ ] **Branch follows naming convention** (feat/, fix/, docs/)
- [ ] **Commit messages are clear**

## Project Structure

```
Xray-test-generation/
├── .claude/
│   └── commands/
│       └── generate-xray-tests/
│           └── command.md           # Main command implementation
│                                    # All logic is here (600+ lines)
├── .github/
│   ├── PULL_REQUEST_TEMPLATE.md   # PR template
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md           # Bug report template
│       └── feature_request.md      # Feature request template
├── docs/
│   ├── GENERATE-XRAY-TESTS-GUIDE.md  # Complete usage guide
│   ├── QUICK-START.md                # Quick start guide
│   ├── INDEX.md                      # Documentation index
│   └── DEVELOPMENT.md                # This file
├── CHANGELOG.md                      # Version history
├── CONTRIBUTING.md                   # Contributing guide
├── README.md                         # Project overview
├── package.json                      # Project metadata & scripts
└── .gitignore                        # Git ignore rules
```

## Command Architecture

The command is implemented as a single markdown file with embedded bash scripts:

```
command.md structure:
├── YAML frontmatter          # Metadata (name, description, mode)
├── Purpose section           # What the command does
├── Prerequisites Check       # Phase 0: Validation
├── Workflow                  # Main execution steps
│   ├── Step 1: Extract ACs
│   ├── Step 2: Format Selection (Interactive)
│   ├── Step 3: Generate Tests (Parallel)
│   └── Step 4: Display Results
├── Command Options           # CLI flags documentation
├── Examples                  # Usage examples
└── Error Handling            # Error scenarios
```

## Common Development Tasks

### Adding a New Command Option

1. **Add to YAML frontmatter**
   ```yaml
   argument-hint: 'TICKET-123 [--format=bdd|manual] [--new-option]'
   ```

2. **Document in "Command Options" section**
   ```markdown
   - `--new-option`: Description of what it does
   ```

3. **Implement in bash workflow**
   ```bash
   if [[ -n "$NEW_OPTION" ]]; then
       # Handle the option
   fi
   ```

4. **Add examples**
   ```bash
   /generate-xray-tests PROJ-123 --new-option
   ```

5. **Test thoroughly**
   - With the option
   - Without the option
   - Combined with other options

### Improving Error Handling

1. **Identify error scenario**
   ```bash
   # What can go wrong?
   - Invalid ticket ID
   - Network failure
   - Missing acceptance criteria
   ```

2. **Add error check**
   ```bash
   if [[ -z "$TICKET_DATA" ]]; then
       echo "❌ Failed to fetch ticket $TICKET"
       echo "   Verify ticket exists and you have access"
       exit 1
   fi
   ```

3. **Test the error path**
   ```bash
   # Force the error
   /generate-xray-tests INVALID-999
   ```

### Optimizing Performance

Current optimizations:
- **Cached ticket fetching** - Single JIRA API call
- **Parallel test creation** - Up to 5 concurrent requests
- **In-memory processing** - No temp files for AC extraction

To add new optimizations:
1. Identify bottleneck (use `time` command)
2. Implement optimization
3. Benchmark before/after
4. Document in "Performance Optimizations" section

## Debugging

### Enable Verbose Output

```bash
# Add to command for debugging
set -x  # Enable bash tracing

# Your code here

set +x  # Disable bash tracing
```

### Check JIRA CLI Connection

```bash
jira me
jira issue view PROJ-123 --plain
```

### Check X-Ray API Access

```bash
curl -v -H "Authorization: Bearer $JIRA_API_TOKEN" \
     "$JIRA_URL/rest/raven/1.0/api/settings"
```

### Common Issues

#### Issue: "Command not found"

**Cause:** Command not properly installed in Claude

**Solution:**
```bash
# Verify .claude directory exists
ls -la ~/your-project/.claude/commands/generate-xray-tests/

# Reinstall if missing
cp -r ~/Xray-test-generation/.claude ~/your-project/
```

#### Issue: "Failed to fetch ticket"

**Cause:** JIRA authentication or connectivity issue

**Solution:**
```bash
# Check JIRA CLI auth
jira me

# Reconfigure if needed
jira init

# Verify environment variables
echo $JIRA_API_TOKEN
echo $JIRA_URL
```

#### Issue: "X-Ray test creation failed"

**Cause:** X-Ray not enabled or permission issues

**Solution:**
```bash
# Check X-Ray access
curl -H "Authorization: Bearer $JIRA_API_TOKEN" \
     "$JIRA_URL/rest/raven/1.0/api/settings"

# Verify permissions:
# - Can you create tests manually in X-Ray?
# - Does your API token have write permissions?
```

#### Issue: "Parallel execution errors"

**Cause:** Rate limiting or too many concurrent requests

**Solution:**
```bash
# Reduce parallel limit in command.md
PARALLEL_LIMIT=3  # Instead of 5
```

## Testing Best Practices

### Use Real Tickets

Always test with actual JIRA tickets that have:
- Clear acceptance criteria
- Multiple ACs (for parallel testing)
- Proper project configuration

### Test Edge Cases

- Empty acceptance criteria
- Very long acceptance criteria
- Special characters in ACs
- Multiple formats of ACs (bullets, numbered, paragraphs)

### Clean Up Test Data

After testing:
```bash
# Delete test X-Ray tests from JIRA
# Don't pollute your project with test data
```

## Release Process

When ready to release a new version:

1. **Update version**
   ```bash
   # In package.json
   "version": "2.3.0"
   ```

2. **Update CHANGELOG.md**
   ```markdown
   ## v2.3.0 (Date)
   - Added: New feature
   - Fixed: Bug fix
   - Changed: Improvement
   ```

3. **Update README.md** (if needed)
   - New features in "What It Does"
   - New examples
   - New documentation links

4. **Create release commit**
   ```bash
   git add .
   git commit -m "chore: Release v2.3.0"
   git tag v2.3.0
   git push origin main --tags
   ```

5. **Create GitHub Release**
   - Go to GitHub Releases
   - Draft new release
   - Copy changelog content
   - Publish

## Getting Help

If you're stuck:

1. **Check existing documentation**
   - README.md
   - This DEVELOPMENT.md
   - docs/GENERATE-XRAY-TESTS-GUIDE.md

2. **Search existing issues**
   - [GitHub Issues](https://github.com/SKSA/Xray-test-generation/issues)

3. **Ask for help**
   - [GitHub Discussions](https://github.com/SKSA/Xray-test-generation/discussions)
   - Create an issue with details

4. **Review similar commands**
   - Look at spec-machine examples
   - Check other Claude commands

## Contributing Guidelines

See [CONTRIBUTING.md](../CONTRIBUTING.md) for:
- Code of conduct
- PR submission process
- Style guidelines
- Review process

---

**Questions?** Open a [discussion](https://github.com/SKSA/Xray-test-generation/discussions) or [issue](https://github.com/SKSA/Xray-test-generation/issues).
