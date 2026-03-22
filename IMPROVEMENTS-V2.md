# AC Workflow - Enhanced Features (v2.0)

> **All requested improvements implemented!**

## 🎉 What's New

You requested these improvements - all are now implemented:

✅ **A. Post Results Back to JIRA**
✅ **B. Confluence Integration**  
✅ **C. Smart AC Detection**
✅ **Multi-Framework Support**
✅ **API Test Generation**
✅ **Figma Integration**
✅ **Smart Selector Scanning**
✅ **Quality Trends Tracking**

---

## 📦 All Commands (v2.0)

### **Core Workflow** (Original)
1. **`/collect-ac`** - Start with ACs (ENHANCED ⭐)
2. **`/generate-e2e-tests`** - Generate tests (ENHANCED ⭐)
3. **`/verify-ac`** - Verify before PR

### **New Commands** (v2.0) 🆕

4. **`/post-to-jira`** - Post verification results to JIRA
5. **`/smart-selector-scan`** - Scan codebase for selectors
6. **`/figma-ac-extractor`** - Extract ACs from Figma designs
7. **`/ac-quality-trends`** - Track quality metrics over time

---

## 🚀 Enhanced `/collect-ac` Features

### **Smart AC Detection** (Multi-Source)

Now pulls ACs from:
- ✅ **JIRA Custom Fields** - Dedicated AC field
- ✅ **JIRA Description** - Multiple formats (BDD, lists, tables)
- ✅ **Confluence Pages** - Linked specs with full parsing
- ✅ **Google Docs** - Specification documents
- ✅ **JIRA Sub-tasks** - Each subtask as an AC
- ✅ **JIRA Comments** - AC clarifications
- ✅ **Figma Designs** - Visual requirements (see `/figma-ac-extractor`)

### **Intelligent Parsing**

Detects and parses:
```
✓ BDD Format (Given/When/Then)
✓ Numbered lists (1. 2. 3.)
✓ Bullet lists (- * •)
✓ Checkbox lists (- [ ])
✓ Tables (| AC | Description |)
✓ Headings ("Acceptance Criteria", "AC:", "DoD")
✓ Custom formats
```

### **Confidence Scoring**

```
⭐⭐⭐⭐⭐ High    - All from structured sources
⭐⭐⭐   Medium  - Mix of structured + manual
⭐       Low     - All manual entry
```

### **Source Linking**

Maintains links back to:
- Original JIRA ticket
- Confluence page URL
- Google Doc link
- Figma design
- Comments and sub-tasks

**Example Output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Detected Acceptance Criteria: EPS-1234
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Sources Detected:
✅ JIRA description
✅ Confluence page: "Email Feature Spec"
✅ Figma design: "Email Flow"
✅ JIRA comments (2)

AC #1: User receives verification email
  Source: JIRA description (line 15)
  Format: User story
  
AC #2: Code expires after 15 minutes
  Source: Confluence (BDD format)
  
AC #3: Button must be #00A862 (Primary Green)
  Source: Figma design "Submit Button"
  Type: Visual requirement

Detection Confidence: ⭐⭐⭐⭐⭐ High
```

---

## 🎭 Enhanced `/generate-e2e-tests` Features

### **Multi-Framework Support**

Generate tests for:

**E2E:**
- ✅ Playwright (recommended)
- ✅ Cypress
- ✅ Selenium

**Component:**
- ✅ React Testing Library
- ✅ Enzyme
- ✅ Vue Test Utils

**Unit:**
- ✅ Jest
- ✅ Vitest
- ✅ Mocha

**API:**
- ✅ Supertest
- ✅ Postman/Newman
- ✅ REST Assured

**Visual:**
- ✅ Percy
- ✅ Chromatic
- ✅ Applitools

**Performance:**
- ✅ Lighthouse
- ✅ WebPageTest

**Accessibility:**
- ✅ Axe-core

### **Smart Test Type Detection**

AI analyzes each AC and suggests appropriate tests:

```javascript
AC: "User receives email"
→ Recommends: E2E, API tests

AC: "Code expires after 15 minutes"  
→ Recommends: Unit, E2E tests

AC: "Button is 44px high"
→ Recommends: Visual regression

AC: "Page accessible to screen readers"
→ Recommends: Accessibility tests
```

### **Complete Test Pyramid**

For each AC, generates:
1. **Unit tests** - Business logic
2. **Component tests** - UI components
3. **Integration tests** - API endpoints
4. **E2E tests** - Full user flows
5. **Visual tests** - Screenshot comparison
6. **A11y tests** - WCAG compliance
7. **Performance tests** - Load times

**Example Generated Tests:**

```
tests/
├── e2e/EPS-1234.spec.ts          (Playwright)
├── api/EPS-1234.test.ts           (Supertest)
├── components/Form.test.tsx       (RTL)
├── unit/expiration.test.ts        (Jest)
├── visual/EPS-1234.visual.ts      (Percy)
├── a11y/EPS-1234.a11y.ts          (Axe)
└── performance/EPS-1234.perf.ts   (Lighthouse)
```

---

## 🆕 New Command: `/post-to-jira`

### **Purpose**
Automatically post AC verification results back to JIRA ticket.

### **Features**

**Posts to JIRA:**
- ✅ AC verification status (passed/failed/needs QA)
- ✅ E2E test results (passed/failed/skipped)
- ✅ Code coverage metrics
- ✅ Test execution time
- ✅ Screenshot/video links
- ✅ CI build links

**Auto-updates:**
- ✅ Ticket status ("Ready for QA")
- ✅ Labels ("ac-verified", "e2e-tested")
- ✅ Mentions assignee

**Example JIRA Comment:**
```
✅ AC Verification Complete - EPS-1234

Verification Date: 2026-03-22
Verified By: mde@hellofresh.com
Branch: feature/EPS-1234

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 Acceptance Criteria Results

| # | Criterion | Status |
|---|-----------|--------|
| 1 | User receives email | ✅ PASSED |
| 2 | Code expires | ✅ PASSED |
| 3 | Success notification | 🔍 Needs QA |

Summary: 2/3 passed, 1 needs QA

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎭 E2E Test Results

Tests: 8 passed, 0 failed
Duration: 45.2s
View Report: [link]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Ready for PR? YES
```

**Usage:**
```bash
/post-to-jira EPS-1234

# With auto-transition
/post-to-jira EPS-1234 --transition "Ready for QA"

# Skip status update
/post-to-jira EPS-1234 --no-transition
```

---

## 🆕 New Command: `/smart-selector-scan`

### **Purpose**
Scan codebase to find all test selectors and create intelligent mapping.

### **What It Finds**

**Selector Types:**
- `data-testid` (React/Web)
- `data-cy` (Cypress)
- `testID` (React Native)
- `aria-label` (Accessibility)
- CSS classes (fallback)

**Additional Intelligence:**
- Component hierarchy
- Route mappings
- API endpoints
- Form fields and validation
- Button/input patterns

### **Generates**

**Selector Map:**
```json
{
  "selectors": {
    "buttons": {
      "submit-button": {
        "file": "src/components/Form.tsx",
        "line": 52,
        "element": "button",
        "text": "Submit",
        "usage": ["VerificationForm", "SignupPage"]
      }
    },
    "inputs": {
      "email-input": {
        "file": "src/components/Form.tsx",
        "line": 45,
        "type": "email",
        "validation": ["required", "email"]
      }
    }
  }
}
```

**Quality Report:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Selector Quality Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Coverage: 87% (342/394 elements)

✅ Good: 95% use data-testid
⚠️ Issues: 52 missing selectors
⚠️ Issues: 8 inconsistent naming
⚠️ Issues: 3 duplicate selectors

Auto-fix available for 45 issues.
```

**Visual Explorer:**
- Interactive HTML map
- Searchable selector list
- Component tree view
- One-click copy selectors

**Integration:**
- Auto-used by `/generate-e2e-tests`
- Generates accurate selectors
- No generic `[data-testid="button"]`
- Uses real: `[data-testid="submit-button"]`

**Usage:**
```bash
# Scan entire codebase
/smart-selector-scan

# Scan specific directory
/smart-selector-scan src/components

# Scan recent changes only
/smart-selector-scan --recent

# Update existing map
/smart-selector-scan --update
```

---

## 🆕 New Command: `/figma-ac-extractor`

### **Purpose**
Extract visual acceptance criteria and design requirements from Figma.

### **What It Extracts**

**Visual Requirements:**
- ✅ Component dimensions (width/height)
- ✅ Colors (hex codes)
- ✅ Typography (font, size, weight)
- ✅ Spacing (padding, margin)
- ✅ Border radius, shadows
- ✅ Component states/variants

**Interaction Requirements:**
- ✅ Button click flows
- ✅ Hover states
- ✅ Loading states
- ✅ Navigation flows

**Accessibility Requirements:**
- ✅ Touch target sizes (44px min)
- ✅ Color contrast ratios (WCAG)
- ✅ Alt text for images
- ✅ Focus states
- ✅ Keyboard navigation

**Designer Notes:**
- ✅ Dev Mode comments
- ✅ Implementation notes
- ✅ Animation specs

### **Example Output**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📐 Extracted from Figma: Email Flow
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎨 Visual ACs (12):
  AC-V1: Email input must be 320px × 48px
  AC-V2: Submit button must be 160px × 48px
  AC-V3: Button background #00A862
  AC-V4: Heading uses Inter Bold 24px
  ...

🖱️ Interaction ACs (4):
  AC-I1: Click Submit → navigate to Verification
  AC-I2: Hover button → background #008C4E
  ...

♿ Accessibility ACs (5):
  AC-A1: Button ≥44px high (touch target)
  AC-A2: Contrast ratio ≥4.5:1 (WCAG 2.1.3)
  ...

💬 Designer Notes (2):
  AC-D1: Use slide-up animation (300ms)
  AC-D2: Loading spinner 24px, primary color
```

### **Auto-Generates Visual Tests**

```typescript
// From Figma specs, generates:
test('AC-V3: Button background color', async ({ page }) => {
  const button = page.locator('[data-testid="submit-button"]');
  const bgColor = await button.evaluate(el => 
    window.getComputedStyle(el).backgroundColor
  );
  
  expect(rgbToHex(bgColor)).toBe('#00A862');
});
```

**Merges with JIRA ACs:**
```
/collect-ac EPS-1234
→ Fetches 4 functional ACs from JIRA

/figma-ac-extractor https://figma.com/...
→ Fetches 12 visual ACs from Figma

Result: 16 total ACs (4 functional + 12 visual)
```

**Usage:**
```bash
/figma-ac-extractor https://figma.com/design/xyz?node-id=1-123

# Auto-merge with existing ACs
/figma-ac-extractor --merge
```

---

## 🆕 New Command: `/ac-quality-trends`

### **Purpose**
Track AC quality metrics over time, identify patterns, measure team performance.

### **Metrics Tracked**

**Coverage:**
- Total tickets/ACs per sprint
- Pass rate over time
- Test coverage (E2E, API, unit, visual)
- Average ACs per ticket

**Quality:**
- AC source quality (JIRA/Confluence/Figma)
- Detection confidence scores
- Multi-source adoption rate

**Velocity:**
- Average time to verify
- Verification rate (tickets/day)
- Time by complexity

**Issues:**
- Total issues per sprint
- Issue categories (selectors, flaky, API, visual)
- Issues per ticket trend

### **Reports Generated**

**Sprint Report:**
```markdown
# AC Quality Report - Sprint 24

Tickets: 18
ACs: 67
Pass Rate: 94% ⬆️ (+5% from last sprint)

Test Coverage:
  E2E: 100% ⬆️
  API: 78% ⬆️
  Visual: 44% ⬆️

Avg Time to Verify: 2.3h ⬇️ (was 3.1h)

Top Issues:
1. Missing selectors: 4
2. Flaky tests: 3
3. API timeouts: 2
```

**Trend Charts:**
```
Pass Rate Trend
100% ┤                         ●
 90% ┤            ●────────●
 80% ┤   ●────●
     └────────────────────────
      S21  S22  S23  S24
```

**Developer Leaderboard:**
```
🏆 Top Performers

| Rank | Developer | ACs | Pass Rate | Sources | Time |
|------|-----------|-----|-----------|---------|------|
| 🥇   | alice@hf  | 24  | 100%      | 3.2     | 2.1h |
| 🥈   | bob@hf    | 18  | 94%       | 2.1     | 2.5h |
| 🥉   | carol@hf  | 15  | 93%       | 1.8     | 1.8h |
```

**Predictive Analytics:**
```
🔮 Next Sprint Forecast

Expected tickets: 19-21
Predicted pass rate: 95-97%
Expected time: 2.0-2.5h
Confidence: HIGH

Risks:
⚠️ Visual test coverage still low

Opportunities:
✨ Figma integration - 56% potential
✨ Visual testing - 56% tickets without
```

**Export Options:**
- Markdown report
- JSON (for dashboards)
- CSV (for spreadsheets)
- Confluence page
- Slack message

**Usage:**
```bash
# Current sprint
/ac-quality-trends

# Specific sprint
/ac-quality-trends --sprint "Sprint 24"

# Compare sprints
/ac-quality-trends --compare "Sprint 23" "Sprint 24"

# Export to Confluence
/ac-quality-trends --export confluence

# JSON for dashboards
/ac-quality-trends --export json
```

---

## 📊 Complete Workflow (v2.0)

```bash
# 1. Start with multi-source ACs
/collect-ac EPS-1234
→ Fetches from JIRA + Confluence
→ Smart detection
→ Creates AC checklist

# 2. Add Figma visual specs
/figma-ac-extractor https://figma.com/...
→ Extracts 12 visual ACs
→ Merges with existing
→ Total: 16 ACs

# 3. Scan for selectors
/smart-selector-scan
→ Finds 342 selectors
→ Builds selector map
→ Quality: 87% coverage

# 4. Generate multi-framework tests
/generate-e2e-tests EPS-1234
→ Uses real selectors
→ Generates: E2E + API + Component + Visual + A11y
→ 15 tests across 5 frameworks

# 5. CODE YOUR FEATURE
# ... implement ...

# 6. Run tests
npx playwright test
npm test

# 7. Verify ACs
/verify-ac EPS-1234
→ Interactive verification
→ Generates report

# 8. Post to JIRA
/post-to-jira EPS-1234
→ Posts results to ticket
→ Updates status to "Ready for QA"
→ Adds labels

# 9. Create PR
gh pr create
→ Link verification report

# 10. Track quality (end of sprint)
/ac-quality-trends
→ Sprint metrics
→ Team performance
→ Trend analysis
```

---

## 📂 Updated File Structure

```
your-repo/
├── .ac-verification/
│   ├── EPS-1234/
│   │   ├── ac-checklist.md          # AC tracking
│   │   ├── sources.json             # Source links
│   │   ├── figma-acs.md             # Figma visual ACs
│   │   ├── figma-screenshots/       # Reference images
│   │   ├── design-tokens.json       # Extracted tokens
│   │   ├── verification-report.md   # Final report
│   │   ├── jira-comment.txt         # Posted comment
│   │   └── test-results.json        # Test outcomes
│   │
│   ├── selectors.json               # Selector map
│   ├── selector-map.html            # Visual explorer
│   ├── selector-quality-report.md   # Quality report
│   │
│   └── reports/
│       ├── sprint-24-report.md      # Sprint metrics
│       ├── sprint-24-metrics.json   # Raw data
│       ├── trends.json              # Historical
│       └── leaderboard.md           # Developer stats
│
└── tests/
    ├── e2e/EPS-1234.spec.ts         # Playwright
    ├── api/EPS-1234.test.ts         # Supertest
    ├── components/Form.test.tsx     # RTL
    ├── unit/expiration.test.ts      # Jest
    ├── visual/EPS-1234.visual.ts    # Percy
    ├── a11y/EPS-1234.a11y.ts        # Axe
    └── performance/EPS-1234.perf.ts # Lighthouse
```

---

## 🎯 Benefits Summary

### **For Developers:**
✅ Multi-source AC detection (less manual entry)
✅ Smart selector mapping (accurate tests)
✅ Multi-framework support (use what you know)
✅ Visual AC extraction (complete requirements)
✅ Automated JIRA updates (less context switching)

### **For Teams:**
📊 Quality metrics tracking
📈 Sprint-over-sprint improvements
🏆 Performance recognition
🔮 Predictive insights
💡 Data-driven decisions

### **For QA:**
✅ Complete AC coverage (functional + visual + a11y)
🎭 Automated test generation
🔍 Know what dev tested
📸 Visual regression tests
♿ Accessibility validation

---

## 📍 Location

All commands stored at:
```
/Users/mde/Documents/spectest/.claude/commands/
├── collect-ac/             (ENHANCED)
├── verify-ac/
├── generate-e2e-tests/     (ENHANCED)
├── post-to-jira/           (NEW)
├── smart-selector-scan/    (NEW)
├── figma-ac-extractor/     (NEW)
└── ac-quality-trends/      (NEW)
```

Documentation at:
```
/Users/mde/Documents/spectest/
├── README.md
├── AC-WORKFLOW-README.md
├── QUICK-START.md
├── INDEX.md
└── IMPROVEMENTS-V2.md      (THIS FILE)
```

---

## ✅ Implementation Status

**All Requested Features: COMPLETE** ✅

✅ A. Post Results Back to JIRA - `/post-to-jira`
✅ B. Confluence Integration - Enhanced `/collect-ac`
✅ C. Smart AC Detection - Enhanced `/collect-ac`
✅ Multi-Framework Support - Enhanced `/generate-e2e-tests`
✅ API Test Generation - Enhanced `/generate-e2e-tests`
✅ Figma Integration - `/figma-ac-extractor`
✅ Smart Selector Scanning - `/smart-selector-scan`
✅ Quality Trends - `/ac-quality-trends`

---

**Ready to use! 🚀**

Start with:
```bash
cd /Users/mde/Documents/spectest
/collect-ac EPS-1234
```
