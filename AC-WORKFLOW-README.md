# AC Verification Workflow for Spec-Machine

> **Developer QA Workflow**: Ensure acceptance criteria are tested before PR merge

---

## 📖 Overview

This workflow adds three custom commands to spec-machine that enforce AC testing before pull requests:

1. **`/collect-ac`** - Initialize task with acceptance criteria
2. **`/generate-e2e-tests`** - Generate Playwright tests from ACs
3. **`/verify-ac`** - Verify AC testing status before PR

## 🎯 The Problem It Solves

**Before:**
- Developers merge PRs without testing all acceptance criteria
- QA finds issues that should have been caught earlier
- No systematic way to track AC testing status
- Manual AC verification is inconsistent

**After:**
- Every AC is explicitly verified before PR
- E2E tests are automatically generated from ACs
- Clear "Ready for PR?" gate based on AC status
- Audit trail of what was tested

---

## 🚀 Quick Start

### Prerequisites

- Spec-machine installed and configured
- (Optional) JIRA CLI for ticket fetching
- (Optional) Playwright for E2E tests

### Installation

The commands are located in:
```
/Users/mde/Documents/spectest/.claude/commands/
├── collect-ac/
│   └── collect-ac.md
├── verify-ac/
│   └── verify-ac.md
└── generate-e2e-tests/
    └── generate-e2e-tests.md
```

To use in your project, copy to your repo:
```bash
cp -r /Users/mde/Documents/spectest/.claude/commands/* your-repo/.claude/commands/
```

Or install via spec-machine config (add to `spec-machine.config.yml`):
```yaml
commands:
  - name: /collect-ac
    path: .claude/commands/collect-ac/collect-ac.md
  - name: /verify-ac
    path: .claude/commands/verify-ac/verify-ac.md
  - name: /generate-e2e-tests
    path: .claude/commands/generate-e2e-tests/generate-e2e-tests.md
```

---

## 📋 Complete Developer Workflow

### Scenario: Developer picks up ticket EPS-1234

```bash
# ========================================
# STEP 1: START TASK
# ========================================

/collect-ac EPS-1234

# What happens:
# - Fetches ticket from JIRA (or asks you to enter manually)
# - Extracts acceptance criteria
# - Creates .ac-verification/EPS-1234/ac-checklist.md
# - Shows summary of ACs

Output:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Task Started: EPS-1234
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Acceptance Criteria: 3
  AC1: User receives verification email
  AC2: Code expires after 15 minutes
  AC3: Clear error messages shown

📝 Next Steps:
  1. Create branch: git checkout -b feature/EPS-1234
  2. Implement the feature
  3. Run: /generate-e2e-tests EPS-1234
  4. Verify: /verify-ac EPS-1234

✅ AC checklist saved to: .ac-verification/EPS-1234/


# ========================================
# STEP 2: CREATE BRANCH & CODE
# ========================================

git checkout -b feature/EPS-1234-email-verification

# ... Developer implements the feature ...
# ... Writes unit tests ...
# ... Tests manually ...


# ========================================
# STEP 3: GENERATE E2E TESTS (Optional)
# ========================================

/generate-e2e-tests EPS-1234

# What happens:
# - Scans your repo to understand tech stack
# - Analyzes your code for routes, components, selectors
# - Generates Playwright test for each AC
# - Creates tests/e2e/EPS-1234.spec.ts

Output:
📊 Repository Analysis:
- Framework: Next.js 14
- Language: TypeScript
- Testing: Playwright configured

🎭 E2E Tests Generated:
✅ tests/e2e/EPS-1234.spec.ts (3 tests)
   - AC #1: User receives verification email
   - AC #2: Code expires after 15 minutes
   - AC #3: Clear error messages shown

🚀 Run tests: npx playwright test


# ========================================
# STEP 4: RUN E2E TESTS
# ========================================

npx playwright test tests/e2e/EPS-1234.spec.ts

# Results:
# ✅ AC #1 test: PASSED
# ✅ AC #2 test: PASSED
# ✅ AC #3 test: PASSED


# ========================================
# STEP 5: VERIFY AC STATUS
# ========================================

/verify-ac EPS-1234

# Interactive Q&A for each AC:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AC #1: User receives verification email
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What's the testing status?
1. ✅ Tested and PASSED
2. ❌ Tested and FAILED
3. 🔍 Needs QA testing
4. N/A Not applicable
5. ⏳ Will test later

Your choice: 1

# Repeat for AC #2, AC #3...

# Final output:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EPS-1234 — Verification Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬────────────────────────────────────┬────────────────┐
│  #  │     Acceptance Criterion           │     Status     │
├─────┼────────────────────────────────────┼────────────────┤
│ 1   │ User receives verification email   │ ✅ PASSED      │
│ 2   │ Code expires after 15 minutes      │ ✅ PASSED      │
│ 3   │ Clear error messages shown         │ ✅ PASSED      │
└─────┴────────────────────────────────────┴────────────────┘

Summary:
- Total AC: 3
- Tested & Passed: 3

Ready for PR? YES ✅

All acceptance criteria verified.


# ========================================
# STEP 6: CREATE PULL REQUEST
# ========================================

gh pr create --title "[EPS-1234] Email verification flow"

# Link verification report in PR description!
```

---

## 📁 File Structure

After running the workflow, you'll have:

```
your-repo/
├── .ac-verification/
│   └── EPS-1234/
│       ├── ac-checklist.md           # AC tracking
│       └── verification-report.md    # Final verification
│
├── tests/
│   └── e2e/
│       ├── EPS-1234.spec.ts         # Generated E2E tests
│       └── EPS-1234-README.md       # Test documentation
│
└── .claude/
    └── commands/
        ├── collect-ac/
        ├── verify-ac/
        └── generate-e2e-tests/
```

---

## 🎯 Command Details

### `/collect-ac [TICKET-ID]`

**Purpose:** Initialize task with acceptance criteria

**Usage:**
```bash
/collect-ac EPS-1234
```

**What it does:**
1. Fetches JIRA ticket (if JIRA CLI available)
2. Extracts or prompts for acceptance criteria
3. Creates AC checklist in `.ac-verification/TICKET-ID/`
4. Optionally triggers `/generate-e2e-tests`

**Output:**
- `.ac-verification/EPS-1234/ac-checklist.md`

**When to use:**
- ✅ At the start of every task
- ✅ Before writing any code
- ✅ To establish clear AC tracking

---

### `/generate-e2e-tests [TICKET-ID]`

**Purpose:** Generate Playwright E2E tests from acceptance criteria

**Usage:**
```bash
/generate-e2e-tests EPS-1234
```

**What it does:**
1. Reads ACs from checklist
2. Scans repo to understand tech stack
3. Analyzes routes, components, selectors
4. Generates Playwright tests for each AC
5. Creates test documentation

**Output:**
- `tests/e2e/EPS-1234.spec.ts` - Test suite
- `tests/e2e/EPS-1234-README.md` - Documentation
- `playwright.config.ts` (if needed)

**When to use:**
- ✅ After coding (recommended) - uses real selectors
- ✅ Before coding - creates test skeleton
- ✅ For regression test suite

**Best practice:**
- Generate AFTER implementing feature for accurate tests
- Review and refine generated tests
- Run tests: `npx playwright test`

---

### `/verify-ac [TICKET-ID]`

**Purpose:** Verify AC testing status before creating PR

**Usage:**
```bash
/verify-ac EPS-1234
```

**What it does:**
1. Reads AC checklist
2. Asks developer about testing status for each AC
3. Updates checklist with status
4. Generates verification report
5. Shows "Ready for PR?" decision

**Testing Status Options:**
- ✅ **Tested and PASSED** - Works correctly
- ❌ **Tested and FAILED** - Found issues
- 🔍 **Needs QA testing** - Requires QA verification
- **N/A Not applicable** - Doesn't apply
- ⏳ **Will test later** - Deferred

**Output:**
- `.ac-verification/EPS-1234/ac-checklist.md` (updated)
- `.ac-verification/EPS-1234/verification-report.md`

**Ready for PR Criteria:**
- All ACs are either:
  - ✅ Tested and PASSED, OR
  - 🔍 Needs QA testing, OR
  - N/A Not applicable
- No ACs are:
  - ❌ Tested and FAILED, OR
  - ⏳ Will test later

**When to use:**
- ✅ Before creating PR
- ✅ After implementing and testing feature
- ✅ To generate verification report for PR

---

## 💡 Best Practices

### For Developers

1. **Always run `/collect-ac` first**
   - Establishes clear AC baseline
   - Creates tracking structure
   - No excuses for missing ACs

2. **Generate E2E tests AFTER coding**
   - More accurate selectors
   - Tests real implementation
   - Catches integration issues

3. **Be honest in `/verify-ac`**
   - Don't mark as "Passed" if untested
   - Use "Needs QA" for complex flows
   - Document issues found

4. **Link verification report in PR**
   - Shows you tested systematically
   - Helps reviewers understand coverage
   - Creates audit trail

5. **Run E2E tests in CI**
   - Add to GitHub Actions workflow
   - Prevents regressions
   - Validates on every push

### For Teams

1. **Make it mandatory**
   - Require verification report in PRs
   - Add to PR template checklist
   - Block merge without verification

2. **Integrate with JIRA**
   - Auto-fetch ACs from tickets
   - Post verification results back to JIRA
   - Link PR to ticket

3. **Track metrics**
   - How many ACs tested per sprint
   - Pass/fail rates
   - Time to verify

4. **Review generated tests**
   - Pair review E2E tests
   - Refine selectors
   - Add edge cases

---

## 🔧 Configuration

### JIRA Integration (Optional)

Install JIRA CLI:
```bash
brew install ankitpokhrel/jira-cli/jira-cli
jira init
```

Configure:
```bash
export JIRA_API_TOKEN="your-token"
```

Commands will auto-fetch tickets.

### Playwright Setup (Optional)

Install Playwright:
```bash
npm init playwright@latest
```

Generated tests will use your existing config.

---

## 📊 Example: Real Ticket

**Ticket:** EPS-1234 - Email Verification Flow

**Acceptance Criteria:**
1. User receives verification email within 30 seconds
2. Verification code expires after 15 minutes
3. User can resend code max 3 times
4. Clear error messages for invalid/expired codes

**Developer Flow:**

```bash
# Start
/collect-ac EPS-1234

# Code the feature
# ... implement email service ...
# ... add expiration logic ...
# ... add rate limiting ...

# Generate tests
/generate-e2e-tests EPS-1234

# Run tests
npx playwright test tests/e2e/EPS-1234.spec.ts
# ✅ All 4 tests passing

# Verify
/verify-ac EPS-1234
# Mark all as "Tested and PASSED"

# Create PR
gh pr create
# Link: .ac-verification/EPS-1234/verification-report.md
```

**Verification Report shows:**
- ✅ AC1: Email sent (tested with E2E + manual)
- ✅ AC2: Expiration works (tested with E2E)
- ✅ AC3: Rate limiting works (tested with unit tests)
- ✅ AC4: Error messages correct (tested with E2E)

**Result:** Clean PR with full AC coverage documented!

---

## 🚨 Troubleshooting

### "No AC checklist found"

**Problem:** Trying to verify before creating checklist

**Solution:** Run `/collect-ac` first

### "JIRA CLI not found"

**Problem:** JIRA integration not configured

**Solution:** Enter ACs manually or install JIRA CLI

### "Generated tests failing"

**Problem:** Tests use wrong selectors or routes

**Solution:**
1. Review generated test file
2. Update selectors to match actual implementation
3. Run in debug mode: `npx playwright test --debug`

### "Can't create PR - verification failed"

**Problem:** Some ACs marked as "Tested and FAILED"

**Solution:**
1. Fix the failing functionality
2. Re-test
3. Run `/verify-ac` again
4. Update status to "Passed"

---

## 🔄 Integration with Existing Workflows

### With Spec-Machine

This workflow integrates seamlessly:

```bash
# Standard spec-machine workflow
/analyze-tech-stack
/train-context
/gather-requirements
/create-spec
/create-tasks

# Add AC workflow
/collect-ac EPS-1234        # ← NEW
/implement-task 1
/implement-task 2
/generate-e2e-tests EPS-1234  # ← NEW
/verify-ac EPS-1234         # ← NEW

# Complete
gh pr create
```

### With CI/CD

Add to `.github/workflows/ci.yml`:

```yaml
name: CI

on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      
      # Run E2E tests
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npx playwright test
      
      # Verify AC coverage
      - name: Check AC verification
        run: |
          # Extract ticket from branch name
          TICKET=$(git branch --show-current | grep -oE '[A-Z]+-[0-9]+')
          
          # Check if verification report exists
          if [ ! -f ".ac-verification/$TICKET/verification-report.md" ]; then
            echo "❌ No AC verification found for $TICKET"
            echo "Run: /verify-ac $TICKET"
            exit 1
          fi
          
          echo "✅ AC verification found"
```

---

## 📈 Benefits

### For Developers
- ✅ Clear checklist of what to test
- ✅ Automated E2E test generation
- ✅ Systematic verification process
- ✅ Documentation for PR reviewers

### For Teams
- 📊 Better quality metrics
- 🎯 Fewer bugs in production
- ⚡ Faster code reviews (clear AC status)
- 📝 Audit trail of testing

### For QA
- 🔍 Know what dev already tested
- 🎯 Focus on complex scenarios
- 📋 Clear acceptance criteria
- ✅ E2E regression suite

---

## 🎓 Advanced Usage

### Custom AC Formats

Edit AC prompts in `/collect-ac` command file to support your format:
- User story format
- BDD Gherkin
- Custom templates

### Multi-Repository

For microservices, track ACs centrally:
```bash
# Shared AC repository
.ac-verification/
  EPS-1234/
    frontend/
    backend/
    api-gateway/
```

### Analytics Dashboard

Parse verification reports for metrics:
```bash
# Count verified ACs per sprint
grep -r "Tested and PASSED" .ac-verification/ | wc -l
```

---

## 📝 Summary

**Three Commands:**
1. `/collect-ac` - Start with ACs
2. `/generate-e2e-tests` - Auto-generate tests
3. `/verify-ac` - Verify before PR

**Key Files:**
- `.ac-verification/TICKET/ac-checklist.md` - AC tracking
- `.ac-verification/TICKET/verification-report.md` - Verification
- `tests/e2e/TICKET.spec.ts` - E2E tests

**Workflow:**
```
Start → Code → Test → Verify → PR
  ↓       ↓      ↓       ↓      ↓
  📋     💻     🎭      ✅     🚀
```

---

## 🤝 Contributing

Location: `/Users/mde/Documents/spectest/`

To improve:
1. Edit command files in `.claude/commands/`
2. Test in spec-machine
3. Submit to main spec-machine repo

---

## 📞 Support

Questions? Check:
- Command files: `.claude/commands/*/`
- This README
- [Spec-Machine Documentation](https://github.com/hellofresh/spec-machine)

---

**Built with ❤️ for better software quality**
