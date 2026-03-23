# Complete Integrated Workflow Guide

## Spec Dev QA Assistant + spec-machine Integration

This guide shows how to use **Spec Dev QA Assistant** together with HelloFresh's **spec-machine** for a complete development workflow.

---

## 🎯 Overview

**Spec Dev QA Assistant** fills gaps in the testing workflow that spec-machine doesn't cover:
- ✅ Multi-source AC collection (JIRA + Confluence + Figma)
- ✅ Automated test generation (Web + Mobile React Native)
- ✅ AC verification with test execution
- ✅ Quality trend tracking

**spec-machine** provides infrastructure:
- ✅ Tech stack analysis
- ✅ Context training
- ✅ Requirements gathering
- ✅ PR creation with JIRA integration

**Together:** Complete workflow from ticket to merged PR with verified ACs.

---

## 🚀 Setup (One-Time Per Repository)

### **Step 1: Install Both Tools**

```bash
# Install spec-machine (if not already installed)
# See: https://github.com/hellofresh/spec-machine

# Install spec-dev-qa-assistant
git clone git@github.com:mde-hf/spec-dev-qa-assistant.git
cd your-project
cp -r ~/spec-dev-qa-assistant/.claude .
# Restart Claude Code
```

### **Step 2: Setup Repository Context**

```bash
# Analyze your tech stack
/analyze-tech-stack
→ Generates: tech-stack.md
→ Used by test generation for better detection

# Train on your repository patterns
/train-context
→ Generates: context-training files
→ Improves code generation quality

# Load context (optional, but recommended)
/load-context-training
```

**You're ready!** This setup only needs to be done once per repository.

---

## 📋 Per-Ticket Workflow

### **Phase 1: Gather Requirements & ACs**

```bash
# 1. Fetch ticket and requirements (spec-machine)
/jira-gather-requirements EPS-1234
```

**What happens:**
- Fetches JIRA ticket details
- May post clarifying questions to ticket
- Gathers initial requirements

**Output:**
- Requirements documented
- Questions posted to JIRA (if needed)

```bash
# 2. Collect test-ready acceptance criteria (your tool)
/collect-ac EPS-1234
```

**What happens:**
- Fetches from JIRA (custom fields, description, sub-tasks, comments)
- Fetches from linked Confluence pages
- Smart AC detection with confidence scoring
- Creates structured AC checklist

**Output:**
```
.ac-verification/EPS-1234/
├── ac-checklist.md          # Test-ready ACs
├── sources.json             # Source links
└── detection-log.txt        # Parsing details
```

```bash
# 3. (Optional) Add visual/interaction ACs from Figma
/figma-ac-extractor https://figma.com/design/xyz123
```

**What happens:**
- Extracts user flows and interactions from Figma
- Focuses on behavior, not visual styling
- Merges with existing ACs

**Output:**
- ACs updated with interaction requirements
- Figma design tokens saved

---

### **Phase 2: Implement Feature**

```bash
# 4. Create branch
git checkout -b feature/EPS-1234-add-login-flow
```

**Follow HelloFresh branch naming:**
- Format: `{type}/{TICKET}-{description}`
- Examples: `feature/EPS-1234-add-login`, `fix/EPS-99-login-bug`

```bash
# 5. Implement your feature
# ... write your code ...
```

**Implementation tips:**
- **For Web:** Add `data-testid` attributes to key elements
  ```jsx
  <button data-testid="submit-button">Submit</button>
  <input data-testid="email-input" />
  ```

- **For React Native:** Add `testID` props
  ```jsx
  <Button testID="submit-button">Submit</Button>
  <TextInput testID="email-input" />
  ```

- Reference your AC checklist as you build
- Mark ACs as you complete them

---

### **Phase 3: Generate & Run Tests**

```bash
# 6. Generate tests based on your implementation
/generate-e2e-tests EPS-1234
```

**Interactive prompts:**
```
Select Platform:
1. 🌐 Web (Browser-based)
2. 📱 Mobile (React Native)
Choose: 1

🔍 Scanning Your Code for Selectors...
✅ Found: 342 selectors in your code (87% coverage)

Select Framework:
1. ✅ Playwright (installed) - Recommended
2. ✅ Jest (installed)
Choose: 1,2
```

**What happens:**
- Scans YOUR code for selectors/testIDs
- Detects framework and tech stack
- Generates tests matching your implementation
- Creates configuration files

**Output:**
```
tests/
├── e2e/
│   └── EPS-1234.spec.ts        # E2E tests
├── unit/
│   └── validation.test.ts       # Unit tests
└── README.md                    # Test docs

# OR for React Native:
.maestro/
└── EPS-1234.yaml               # Maestro E2E tests
```

```bash
# 7. Run tests
npx playwright test              # Web
# or
maestro test .maestro/EPS-1234.yaml  # React Native
```

**Fix failures:**
- Update implementation
- Add missing selectors/testIDs
- Re-run tests until green

---

### **Phase 4: Verify Acceptance Criteria**

```bash
# 8. Verify all ACs interactively
/verify-ac EPS-1234
```

**Interactive flow:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AC #1: User can login with valid credentials
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

How did this go?
1. ✅ Tested and PASSED
2. ❌ Tested and FAILED
3. 🔍 Needs QA testing
4. N/A Not applicable
5. ⏳ Will test later

Choose: 1

[Repeat for all ACs...]

✅ Verification Complete!

Post results to JIRA ticket EPS-1234? (Y/n): y
→ ✅ Posted to JIRA: [link]
```

**What happens:**
- Step through each AC
- Record test status
- Track issues for failed ACs
- Generate verification report
- Optionally post results to JIRA

**Output:**
```
.ac-verification/EPS-1234/
├── verification-report.md       # Full report
└── jira-comment.txt            # JIRA comment (if posted)
```

---

### **Phase 5: Create Pull Request**

```bash
# 9. Commit your changes
git add .
git commit -m "feat: Add user login flow

Implements email/password authentication with:
- Form validation
- Error handling
- Loading states
- E2E test coverage

AC Verification: 5/5 passed

[Cursor_Code]"

git push -u origin feature/EPS-1234-add-login-flow
```

```bash
# 10. Create PR with full context (spec-machine)
/create-pr
```

**What happens:**
- Detects JIRA ticket from branch name
- Links ticket in PR description
- Uses repo's PR template
- Auto-applies labels (enhancement, typescript, EPS, etc.)
- Follows HelloFresh PR conventions

**Result: Perfect PR**
```markdown
# [EPS-1234]: Add user login flow

## JIRA Ticket
https://jira.hellofresh.com/browse/EPS-1234

## Description
Implements user login flow with email/password authentication.
Includes form validation, error handling, and loading states.

## Type of Change
- [x] New feature

## Testing
- [x] E2E tests added (5 tests)
- [x] Unit tests added (8 tests)
- [x] AC verification: 5/5 passed ✅

## Checklist
- [x] Code follows style guidelines
- [x] Self-review completed
- [x] Tests added and passing
- [x] Documentation updated

Labels: enhancement, typescript, EPS, needs-review
```

---

### **Phase 6: Track Quality (End of Sprint)**

```bash
# 11. Generate quality report
/ac-quality-trends --sprint "Sprint 24"
```

**Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AC Quality Report - Sprint 24
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Overview
Tickets: 18 completed
ACs: 67 total
Pass Rate: 94% ⬆️ (+5%)

⚡ Velocity
Avg Time to Verify: 2.3h ⬇️ (was 3.1h)

🐛 Top Issues
1. Missing selectors: 4
2. Flaky tests: 3

🏆 Top Performers
1. alice@hf.com - 100% pass rate
2. david@hf.com - 95% pass rate
```

---

## 🎯 Quick Reference

### **One-Time Setup:**
```bash
/analyze-tech-stack && /train-context
```

### **Per-Ticket (5 Commands):**
```bash
/jira-gather-requirements EPS-1234  # 1. Requirements
/collect-ac EPS-1234                # 2. ACs
# Implement code                    # 3. Build
/generate-e2e-tests EPS-1234        # 4. Tests
/verify-ac EPS-1234                 # 5. Verify
/create-pr                          # 6. PR
```

---

## 💡 Pro Tips

### **1. Use Both Tools Together**
- spec-machine: Infrastructure (requirements, PRs, design system)
- Your tool: Testing workflow (ACs, test gen, verification, quality)

### **2. Add Selectors Early**
- Add `data-testid` / `testID` as you code
- Better test generation
- Higher selector coverage

### **3. Verify Before PR**
- Always run `/verify-ac` before creating PR
- Catches issues early
- Complete verification in PR description

### **4. Track Quality**
- Run `/ac-quality-trends` at sprint end
- Identify patterns and improvements
- Celebrate wins with team

### **5. Optional: Check Accessibility**
```bash
/collect-ac EPS-1234
/a11y-check https://figma.com/...    # spec-machine
/figma-ac-extractor https://...      # Your tool
```

---

## 📊 Time Savings

**Per Ticket:**
- Requirements gathering: -30 min (automated)
- Test writing: -90 min (generated)
- AC verification: -20 min (structured)
- PR creation: -10 min (automated)

**Total: ~2.5 hours saved per ticket**

**Per Sprint (20 tickets):**
- **50 hours saved team-wide**

---

## 🆘 Troubleshooting

### **Issue: `/create-pr` not found**
```bash
# Make sure spec-machine is installed
spec-machine --version
```

### **Issue: Low selector coverage**
```bash
# Add more test selectors to your code
<button data-testid="submit-btn">     # Web
<Button testID="submit-btn">          # React Native
```

### **Issue: Tests failing**
```bash
# Re-generate with better selectors
/generate-e2e-tests EPS-1234
```

---

## 📚 Documentation

- **Spec Dev QA Assistant:** This repo's README.md
- **spec-machine:** https://github.com/hellofresh/spec-machine
- **Playwright:** https://playwright.dev
- **Maestro:** https://maestro.mobile.dev

---

**Happy Testing!** 🚀
