# Spec Dev QA Assistant

> Your AI-powered QA assistant for spec-machine and Claude Code

Automate your entire QA workflow from acceptance criteria to verified tests with JIRA integration and quality tracking.

---

## 🎯 What It Does

**Spec Dev QA Assistant** streamlines your testing workflow by automating:

- ✅ **Multi-source AC Detection** - Automatically fetch ACs from JIRA, Confluence, and Figma
- ✅ **Automated Test Generation** - Generate E2E, API, and unit tests from acceptance criteria
- ✅ **Platform Support** - Works with Web (Playwright, Cypress) and Mobile React Native (Maestro, Detox)
- ✅ **Smart Selector Scanning** - Map test selectors across your codebase
- ✅ **Interactive Verification** - Step-by-step AC verification with test execution
- ✅ **JIRA Integration** - Post verification results directly to tickets
- ✅ **Quality Tracking** - Track team performance and identify trends over time
- ✅ **spec-machine Integration** - Works seamlessly with HelloFresh's spec-machine for complete workflow

---

## 🚀 Quick Start (5 minutes)

### 1. Clone the Repository

```bash
git clone git@github.com:mde-hf/spec-dev-qa-assistant.git
cd spec-dev-qa-assistant
```

### 2. Install to Your Project

```bash
cd ~/your-project
cp -r ~/spec-dev-qa-assistant/.claude .
```

### 3. Restart Claude Code

Restart Claude Code to load the new commands.

### 4. Try It Out

```bash
# In Claude Code chat
/collect-ac EPS-1234
```

That's it! 🎉

---

## 📋 Available Commands

| Command | Description |
|---------|-------------|
| `/collect-ac` | Collect acceptance criteria from JIRA, Confluence, and Figma |
| `/create-xray-tests` | Generate test cases in X-Ray from ACs (BDD or Manual format) |
| `/generate-e2e-tests` | Generate automated tests (Web: Playwright/Cypress, Mobile: Maestro/Detox) |
| `/verify-ac` | Interactively verify each acceptance criterion |
| `/post-to-jira` | Post verification results to JIRA ticket (auto-prompted after verify) |
| `/figma-ac-extractor` | Extract user flow ACs from Figma designs |
| `/smart-selector-scan` | Scan codebase for test selectors (auto-runs with generate-e2e-tests) |
| `/ac-quality-trends` | Track quality metrics across sprints |

### **Integrates with spec-machine:**

| spec-machine Command | When to Use |
|---------------------|-------------|
| `/analyze-tech-stack` | One-time setup: Analyze project dependencies |
| `/train-context` | One-time setup: Train on repo patterns |
| `/jira-gather-requirements` | Before `/collect-ac`: Fetch ticket requirements |
| `/create-pr` | After `/verify-ac`: Create PR with JIRA link & labels |
| `/a11y-check` | Optional: Check Figma design accessibility |

---

## 🔄 Complete Workflow

### **Prerequisites (One-Time Setup)**

```bash
# Setup spec-machine context (if using spec-machine)
/analyze-tech-stack         # Analyze project dependencies
/train-context              # Train on your repo patterns
```

### **Per-Ticket Workflow**

```bash
# 1️⃣ Gather requirements and acceptance criteria
/jira-gather-requirements EPS-1234  # spec-machine: Fetch ticket & requirements
/collect-ac EPS-1234                # Collect ACs from JIRA + Confluence + Figma
→ Fetches ACs from all sources
→ Smart detection with confidence scoring
→ Creates AC checklist

# 2️⃣ (Optional) Add visual ACs from Figma
/figma-ac-extractor https://figma.com/design/xyz
→ Extracts user flows and interactions
→ Merges with existing ACs

# 3️⃣ (Optional) Create X-Ray test cases
/create-xray-tests EPS-1234
→ Choose format: BDD or Manual Test Steps
→ Generate test cases from ACs
→ Create X-Ray Test issues in JIRA
→ Link tests to story

# 4️⃣ Create branch (if not already)
git checkout -b feature/EPS-1234-description

# 5️⃣ Implement your feature
# ... write your code ...
# Build the feature according to ACs

# 6️⃣ Generate tests for your implementation
/generate-e2e-tests EPS-1234
→ Select platform: Web or Mobile React Native
→ Auto scans for selectors in your code
→ Choose framework (Playwright/Maestro/etc.)
→ Generates tests based on your implementation + ACs

# 7️⃣ Run tests
npx playwright test                     # Web
# or
maestro test .maestro/EPS-1234.yaml     # React Native

# 8️⃣ Verify ACs
/verify-ac EPS-1234
→ Interactive verification
→ Test status tracking
→ Asks: "Post to JIRA?" → Auto-posts results

# 9️⃣ Create PR with full context
/create-pr  # spec-machine: Creates PR with JIRA link, labels, template
→ Links JIRA ticket automatically
→ Uses repo's PR template
→ Applies correct labels
→ Follows team conventions

# 🔟 (End of sprint) Track quality
/ac-quality-trends --sprint "Sprint 24"
→ See pass rates, velocity, issues
→ Developer leaderboard
→ Recommendations
```

---

## 🎯 Key Features

### **Multi-Source AC Detection**

Automatically fetches ACs from:
- **JIRA** - Custom fields, description, sub-tasks, comments
- **Confluence** - Linked pages with smart parsing
- **Figma** - User flows, interactions, form behaviors

Smart parsing handles:
- BDD format (Given/When/Then)
- Bullet lists
- Numbered lists
- Tables
- Checkboxes

### **Framework Flexibility**

Generates tests for:
- **E2E:** Playwright, Cypress
- **Unit/Integration:** Jest, Vitest, Mocha
- **Component:** React Testing Library, Vue Test Utils
- **API:** Supertest, REST Assured
- **Performance:** Lighthouse
- **Accessibility:** Axe-core

### **Smart Selector Scanning**

Automatically scans for:
- `data-testid`
- `data-cy`
- `testID` (React Native)
- `aria-label`
- CSS classes (fallback)

Generates quality report with auto-fix suggestions.

### **JIRA Integration**

Posts formatted comments with:
- AC status (passed/failed/needs QA)
- Test results (E2E, API, unit)
- Code coverage metrics
- Test evidence
- Links to reports

Auto-updates ticket status and labels.

### **Quality Tracking**

Tracks across sprints:
- Pass rates
- Test coverage by type
- Time to verify
- Issue patterns
- Source quality
- Developer performance

---

## 📦 What's Included

### Commands
- `collect-ac/` - Multi-source AC fetching
- `verify-ac/` - Interactive verification
- `generate-e2e-tests/` - Test generation
- `post-to-jira/` - JIRA integration
- `figma-ac-extractor/` - Figma integration
- `smart-selector-scan/` - Selector mapping
- `ac-quality-trends/` - Quality metrics

### Documentation
- `README.md` - This file
- `CHANGELOG.md` - Version history and updates
- `docs/QUICK-START.md` - 5-minute guide
- `docs/AC-WORKFLOW-README.md` - Complete workflow guide
- `docs/IMPROVEMENTS-V2.md` - Feature descriptions
- `docs/FIGMA-AC-FOCUS.md` - Figma usage guide
- `docs/INDEX.md` - Navigation guide
- `docs/HOW-TO-SHARE.md` - Sharing guide

---

## 🔧 Prerequisites

### Required
- **Claude Code** - AI coding assistant
- **JIRA CLI** - For JIRA integration
  ```bash
  # Install if needed
  brew install ankitpokhrel/jira-cli/jira-cli
  jira init
  ```

### Optional
- **Figma MCP** - For Figma integration
  ```bash
  # In Claude Code chat
  /add-plugin figma
  ```
- **Playwright** - For E2E test generation
  ```bash
  npm install -D @playwright/test
  ```

---

## 📚 Documentation

- **[Quick Start](docs/QUICK-START.md)** - Get started in 5 minutes
- **[Complete Workflow Guide](docs/AC-WORKFLOW-README.md)** - Detailed workflow documentation
- **[Latest Updates](CHANGELOG.md)** - What's new in v2.1
- **[Feature Descriptions](docs/IMPROVEMENTS-V2.md)** - Deep dive into features
- **[Figma Integration](docs/FIGMA-AC-FOCUS.md)** - How Figma extraction works
- **[Full Index](docs/INDEX.md)** - Navigate all documentation

---

## 💡 Examples

### Example 1: Simple Feature

```bash
/collect-ac EPS-5678
→ Found 3 ACs from JIRA

# Implement the feature
# ... write code to satisfy the ACs ...

/generate-e2e-tests EPS-5678
→ Scans your implementation
→ Select framework: 1 (Playwright)
→ Generated 3 E2E tests based on your code

# Run tests
npx playwright test

/verify-ac EPS-5678
→ All passed ✅
→ Post to JIRA? Yes
→ Posted to EPS-5678
```

### Example 2: Complex Flow with Figma

```bash
/collect-ac EPS-9012
→ Found 5 ACs from JIRA + Confluence

/figma-ac-extractor https://figma.com/design/abc123
→ Added 4 interaction ACs from Figma
→ Total: 9 ACs

# Implement the feature
# ... write code for all 9 ACs ...

/generate-e2e-tests EPS-9012
→ Scanned selectors: 87% coverage in your code
→ Generated 9 tests (Playwright + Jest)

# Run tests
npx playwright test

/verify-ac EPS-9012
→ 8 passed, 1 needs QA
→ Posted to JIRA
```

### Example 3: Sprint Retrospective

```bash
/ac-quality-trends --sprint "Sprint 24"

→ 18 tickets completed
→ 94% pass rate ⬆️ (+5%)
→ 2.3h avg verification time ⬇️
→ Top issues: Missing selectors (4)
→ Recommendations: Increase visual testing
```

---

## 🎯 Benefits

### For Developers
- ⚡ **2+ hours saved** per ticket
- 🤖 **Automated test generation** - No more manual test writing
- 🎯 **Clear acceptance criteria** - Know exactly what to build
- ✅ **Confidence** - Tests written before implementation

### For QA
- 📋 **Structured verification** - Step-by-step checklist
- 🔍 **Complete coverage** - E2E, API, unit, visual tests
- 📊 **Quality metrics** - Track team performance
- 🔄 **Automated updates** - Results post to JIRA automatically

### For Teams
- 📈 **Visibility** - Quality trends and velocity tracking
- 🏆 **Accountability** - Developer leaderboards
- 🎯 **Process improvement** - Identify bottlenecks
- 🤝 **Consistency** - Standardized workflow

---

## 🔄 Updates

### v2.1 (March 22, 2026)
- ✅ JIRA post now auto-prompted after verify-ac
- ✅ Framework selection simplified (shows detected only)
- ✅ Smart selector scan integrated into generate-e2e-tests
- ✅ Figma extractor focuses on user flows (not visual styling)

See [WORKFLOW-UPDATES-V2.1.md](WORKFLOW-UPDATES-V2.1.md) for full details.

---

## 🤝 Contributing

Found a bug? Have a suggestion? Want to add a feature?

1. Create an issue
2. Submit a PR
3. Share feedback in #spec-dev-qa-assistant
