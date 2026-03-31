---
name: /start-qa-verification
argument-hint: '[PR-NUMBER or PR-URL]'
description: Automated PR QA verification - Extract ACs, generate Cypress tests, run and report results
mode: single-agent
dependencies:
  - GitHub CLI (gh)
  - JIRA CLI (jira) (optional)
  - Cypress (auto-installs if missing)
---

# Start QA Verification

## Purpose
Fully automated PR verification workflow: Extract acceptance criteria from PR/JIRA, generate Cypress E2E tests, run them headfully, and display pass/fail results in a table.

## Workflow

### Step 1: Get PR Input

Ask user: "Enter PR number or URL:"

```
Examples:
  • PR number: 123
  • PR URL: https://github.com/hellofresh/web/pull/123
  • Current branch: (leave empty to auto-detect from current branch)
```

**Parse input:**
```javascript
function parsePRInput(input) {
  // Extract PR number from URL
  if (input.includes('github.com')) {
    const match = input.match(/\/pull\/(\d+)/);
    return match ? match[1] : null;
  }
  
  // Direct number
  if (/^\d+$/.test(input)) {
    return input;
  }
  
  // Empty = detect from current branch
  if (!input) {
    return 'auto-detect';
  }
  
  return null;
}
```

Store as `$PR_NUMBER`

### Step 2: Fetch PR Details

**If auto-detect:**
```bash
# Get current branch
CURRENT_BRANCH=$(git branch --show-current)

# Find PR for current branch
PR_NUMBER=$(gh pr list --head "$CURRENT_BRANCH" --json number --jq '.[0].number')

if [ -z "$PR_NUMBER" ]; then
  echo "❌ No PR found for current branch: $CURRENT_BRANCH"
  exit 1
fi
```

**Fetch PR information:**
```bash
gh pr view $PR_NUMBER --json number,title,body,headRefName,baseRefName,files,additions,deletions
```

Store:
- `$PR_TITLE`
- `$PR_BODY`
- `$PR_BRANCH` (head branch)
- `$BASE_BRANCH` (base branch)
- `$FILES_CHANGED` (list of modified files)

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PR Information
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PR #123: Add shopping cart feature
Branch: feature/CART-456-shopping-cart
Files changed: 12 files (+234, -56)
```

### Step 3: Checkout PR Branch

**Check if already on PR branch:**
```bash
CURRENT_BRANCH=$(git branch --show-current)

if [ "$CURRENT_BRANCH" != "$PR_BRANCH" ]; then
  echo "🔄 Switching to PR branch: $PR_BRANCH"
  
  # Fetch latest
  git fetch origin "$PR_BRANCH"
  
  # Check if branch exists locally
  if git show-ref --verify --quiet "refs/heads/$PR_BRANCH"; then
    # Branch exists, checkout and pull
    git checkout "$PR_BRANCH"
    git pull origin "$PR_BRANCH"
  else
    # Branch doesn't exist locally, create from remote
    git checkout -b "$PR_BRANCH" "origin/$PR_BRANCH"
  fi
  
  echo "✅ Now on branch: $PR_BRANCH"
else
  echo "✅ Already on PR branch: $PR_BRANCH"
fi
```

### Step 4: Extract Acceptance Criteria (Priority Order)

#### Priority 1: PR Description

Parse PR body for ACs:

```javascript
function extractACsFromPR(prBody) {
  const acs = [];
  
  // Look for AC sections
  const acPatterns = [
    /#{1,3}\s*Acceptance Criteria:?\s*\n([\s\S]*?)(?=\n#{1,3}|\n---|\Z)/i,
    /#{1,3}\s*AC:?\s*\n([\s\S]*?)(?=\n#{1,3}|\n---|\Z)/i,
    /#{1,3}\s*Definition of Done:?\s*\n([\s\S]*?)(?=\n#{1,3}|\n---|\Z)/i,
  ];
  
  for (const pattern of acPatterns) {
    const match = prBody.match(pattern);
    if (match) {
      const acText = match[1];
      
      // Parse bullet points or numbered lists
      const lines = acText.split('\n');
      lines.forEach(line => {
        const trimmed = line.trim();
        // Match: - [ ] AC, - AC, * AC, 1. AC, etc.
        if (trimmed.match(/^[-*]\s*(\[.\])?\s*.+/) || trimmed.match(/^\d+\.\s*.+/)) {
          const ac = trimmed.replace(/^[-*]\s*(\[.\])?\s*/, '').replace(/^\d+\.\s*/, '');
          if (ac.length > 0) {
            acs.push({ text: ac, source: 'PR', checked: false });
          }
        }
      });
      break;
    }
  }
  
  return acs;
}
```

#### Priority 2: JIRA Ticket

Extract JIRA ticket from PR:
```bash
# Try to extract from PR title
JIRA_TICKET=$(echo "$PR_TITLE" | grep -oE '[A-Z]+-[0-9]+' | head -1)

# If not in title, try branch name
if [ -z "$JIRA_TICKET" ]; then
  JIRA_TICKET=$(echo "$PR_BRANCH" | grep -oE '[A-Z]+-[0-9]+' | head -1)
fi

# If not in branch, try PR body
if [ -z "$JIRA_TICKET" ]; then
  JIRA_TICKET=$(echo "$PR_BODY" | grep -oE '[A-Z]+-[0-9]+' | head -1)
fi
```

If JIRA ticket found:
```bash
if [ -n "$JIRA_TICKET" ]; then
  echo "🔍 Found JIRA ticket: $JIRA_TICKET"
  
  # Fetch ticket details
  jira issue view "$JIRA_TICKET" --plain > /tmp/jira-ticket.txt
  
  # Extract ACs from custom field or description
  # Look for "Acceptance Criteria" field
fi
```

Parse JIRA ACs using same logic as `/collect-ac` command.

#### Priority 3: Story (Parent Ticket)

```bash
# Get parent link from JIRA ticket
PARENT_KEY=$(jira issue view "$JIRA_TICKET" --plain | grep "Parent:" | awk '{print $2}')

if [ -n "$PARENT_KEY" ]; then
  echo "📖 Found parent story: $PARENT_KEY"
  jira issue view "$PARENT_KEY" --plain > /tmp/jira-story.txt
  # Extract ACs from story
fi
```

#### Priority 4: Epic

```bash
# Get epic link
EPIC_KEY=$(jira issue view "$JIRA_TICKET" --plain | grep "Epic:" | awk '{print $2}')

if [ -n "$EPIC_KEY" ]; then
  echo "📚 Found epic: $EPIC_KEY"
  jira issue view "$EPIC_KEY" --plain > /tmp/jira-epic.txt
  # Extract ACs from epic
fi
```

**Display all found ACs:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Acceptance Criteria Found
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

From PR (#123):
  1. User can add items to cart
  2. Cart total updates correctly
  
From JIRA (CART-456):
  3. Checkout button is enabled when cart has items
  4. Empty cart shows appropriate message

From Story (CART-100):
  5. Cart persists across page refreshes

Total: 5 acceptance criteria
```

### Step 5: Scan PR for Selectors

**Scan modified files for test selectors:**

```bash
# Get list of modified component/page files
MODIFIED_FILES=$(gh pr view $PR_NUMBER --json files --jq '.files[].path' | grep -E '\.(tsx?|jsx?)$')

echo "🔍 Scanning modified files for selectors..."

# Scan for data-testid attributes
rg "data-testid=['\"]([^'\"]+)['\"]" $MODIFIED_FILES -o -r '$1' > /tmp/pr-selectors.txt

# Scan for data-test-id attributes
rg "data-test-id=['\"]([^'\"]+)['\"]" $MODIFIED_FILES -o -r '$1' >> /tmp/pr-selectors.txt

# Count unique selectors
SELECTOR_COUNT=$(sort /tmp/pr-selectors.txt | uniq | wc -l)

echo "✅ Found $SELECTOR_COUNT unique test selectors"
```

**Display selector map:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Test Selectors Found in PR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File: components/ShoppingCart.tsx
  • cart-add-button
  • cart-item-quantity
  • cart-total-price
  • checkout-button

File: pages/checkout.tsx
  • checkout-form
  • payment-submit

Total: 12 selectors detected
```

**If no selectors found:**
```
⚠️ No test IDs found in modified files.

Options:
  1. Continue with text/role selectors (less reliable)
  2. Cancel and add test IDs first

Choose option (1 or 2):
```

### Step 6: Generate Cypress Tests

**Create test file:**

File: `cypress/e2e/pr-${PR_NUMBER}-verification.cy.ts`

```typescript
describe('PR #${PR_NUMBER} Verification', () => {
  beforeEach(() => {
    // Setup: Visit the relevant page
    cy.visit('/cart'); // Infer from modified files
  });

  // Generate test for each AC
  ${acs.map((ac, index) => `
  it('AC ${index + 1}: ${ac.text}', () => {
    // Map AC to test steps based on:
    // - Available selectors from PR
    // - AC text keywords (add, update, click, show, etc.)
    // - Modified files context
    
    ${generateTestStepsForAC(ac, selectors)}
  });
  `).join('\n')}
});
```

**Smart test generation logic:**

```javascript
function generateTestStepsForAC(ac, selectors) {
  const steps = [];
  const acText = ac.text.toLowerCase();
  
  // Detect action keywords
  if (acText.includes('add') || acText.includes('click')) {
    const buttonSelector = selectors.find(s => s.includes('button') || s.includes('add'));
    if (buttonSelector) {
      steps.push(`cy.get('[data-testid="${buttonSelector}"]').click();`);
    }
  }
  
  if (acText.includes('total') || acText.includes('price')) {
    const priceSelector = selectors.find(s => s.includes('total') || s.includes('price'));
    if (priceSelector) {
      steps.push(`cy.get('[data-testid="${priceSelector}"]').should('be.visible');`);
    }
  }
  
  if (acText.includes('shows') || acText.includes('displays') || acText.includes('message')) {
    // Add assertion for visibility
    steps.push(`cy.contains('${extractExpectedText(acText)}').should('be.visible');`);
  }
  
  return steps.join('\n    ');
}
```

**Display generated test:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Test Generated
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File: cypress/e2e/pr-123-verification.cy.ts
Tests: 5 test cases (one per AC)
Lines: 78

Preview:
  ✓ AC 1: User can add items to cart
  ✓ AC 2: Cart total updates correctly
  ✓ AC 3: Checkout button is enabled
  ✓ AC 4: Empty cart shows message
  ✓ AC 5: Cart persists across refreshes
```

### Step 7: Check/Install Cypress

```bash
# Check if Cypress is installed
if ! npm list cypress > /dev/null 2>&1; then
  echo "📦 Cypress not found. Installing..."
  
  npm install --save-dev cypress
  
  # Initialize Cypress if needed
  if [ ! -d "cypress" ]; then
    npx cypress open --e2e
  fi
  
  echo "✅ Cypress installed"
else
  echo "✅ Cypress already installed"
fi
```

### Step 8: Run Cypress Tests (Headful Mode)

```bash
echo "🧪 Running Cypress tests in headful mode..."
echo ""
echo "Opening Cypress Test Runner..."
echo "Please run the test: pr-${PR_NUMBER}-verification.cy.ts"
echo ""

# Open Cypress in headed mode
npx cypress open --e2e --config specPattern="cypress/e2e/pr-${PR_NUMBER}-verification.cy.ts"
```

**Wait for user to complete tests:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 Cypress Test Runner Opened
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. The Cypress UI should be open
2. Select the test: pr-123-verification.cy.ts
3. Watch the tests run
4. When complete, return here

Press Enter when tests are finished...
```

**Read test results:**
```bash
# Parse Cypress results from latest run
CYPRESS_RESULTS=$(cat cypress/results/*.json 2>/dev/null || echo "{}")

# Or parse from terminal output
# Count passed/failed from test runner
```

### Step 9: Display Results Table

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 QA Verification Results - PR #123
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬───────────────────────────────────────┬────────┬──────────┬──────────────┐
│ AC# │ Acceptance Criterion                  │ Source │ Status   │ Duration     │
├─────┼───────────────────────────────────────┼────────┼──────────┼──────────────┤
│ 1   │ User can add items to cart           │ PR     │ ✅ PASS  │ 2.3s         │
│ 2   │ Cart total updates correctly          │ PR     │ ❌ FAIL  │ 1.8s         │
│ 3   │ Checkout button is enabled           │ JIRA   │ ✅ PASS  │ 1.2s         │
│ 4   │ Empty cart shows appropriate message  │ JIRA   │ ✅ PASS  │ 0.9s         │
│ 5   │ Cart persists across page refreshes   │ Story  │ ⏭️  SKIP  │ N/A          │
└─────┴───────────────────────────────────────┴────────┴──────────┴──────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📈 Summary:
  Total ACs: 5
  ✅ Passed: 3 (60%)
  ❌ Failed: 1 (20%)
  ⏭️  Skipped: 1 (20%)
  
  Total Duration: 6.2s

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

❌ Failed Tests:

AC #2: Cart total updates correctly
  Error: Expected "$10.00" but got "$0.00"
  File: cypress/e2e/pr-123-verification.cy.ts:18
  Screenshot: cypress/screenshots/pr-123/ac-2-fail.png

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📁 Files Generated:
  ✓ cypress/e2e/pr-123-verification.cy.ts (kept)
  ✓ cypress/screenshots/pr-123/*.png
  ✓ cypress/videos/pr-123/*.mp4

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next Steps:
  • Fix failing tests and re-run: /start-qa-verification 123
  • Post results to PR: /post-to-jira CART-456
  • Continue with manual QA for skipped ACs
```

### Step 10: Offer Next Actions

```
What would you like to do next?

1. 📝 Add comment to PR with results
2. 🔄 Re-run verification after fixes
3. 📋 Post results to JIRA ticket
4. ✅ Exit (files are saved)

Choose option (1-4):
```

**If user selects 1:**
```bash
# Create markdown comment for PR
COMMENT="## QA Verification Results

✅ **Passed:** 3/5 (60%)
❌ **Failed:** 1/5 (20%)

### Test Results

| AC# | Criterion | Status |
|-----|-----------|--------|
| 1 | User can add items to cart | ✅ PASS |
| 2 | Cart total updates correctly | ❌ FAIL |
| 3 | Checkout button is enabled | ✅ PASS |
| 4 | Empty cart shows message | ✅ PASS |
| 5 | Cart persists across refreshes | ⏭️ SKIP |

### Failed Tests

**AC #2:** Cart total updates correctly
- Expected: \$10.00
- Got: \$0.00

Test file: \`cypress/e2e/pr-123-verification.cy.ts\`
"

# Post comment to PR
gh pr comment $PR_NUMBER --body "$COMMENT"

echo "✅ Comment posted to PR #$PR_NUMBER"
```

## Error Handling

| Issue | Solution |
|-------|----------|
| PR not found | Show error with available PRs: `gh pr list` |
| Branch checkout fails | Show git status, ask user to resolve conflicts |
| No ACs found | Warn user, offer to skip or cancel |
| Cypress installation fails | Show npm error, suggest manual install |
| Test generation fails | Show error, save partial tests |
| Test run fails | Capture error, show in results table |

## Prerequisites

```bash
# Required
✓ GitHub CLI: gh --version
✓ Git repository with remote

# Optional
✓ JIRA CLI: jira --version (for JIRA AC extraction)

# Auto-installed if missing
- Cypress
```

## File Outputs

```
cypress/
├── e2e/
│   └── pr-123-verification.cy.ts     # Generated test file (kept)
├── screenshots/
│   └── pr-123/                       # Test screenshots
│       └── ac-2-fail.png
└── videos/
    └── pr-123/                       # Test recordings
        └── verification.mp4
```

## Usage

Run this command when:
- ✅ PR is ready for QA verification
- ✅ You want automated AC testing
- ✅ Before requesting PR review
- ✅ To verify all ACs are covered

## Example Session

```bash
$ /start-qa-verification 123

📋 PR #123: Add shopping cart feature
🔄 Checking out branch: feature/CART-456-shopping-cart
✅ Branch checked out

📝 Extracting acceptance criteria...
  ✓ Found 2 ACs in PR description
  ✓ Found 2 ACs in JIRA ticket CART-456
  ✓ Found 1 AC in Story CART-100
  Total: 5 ACs

🎯 Scanning for test selectors...
  ✓ Found 12 selectors in modified files

✅ Generated test: cypress/e2e/pr-123-verification.cy.ts

🧪 Running Cypress headful mode...
  [Cypress UI opens]
  
📊 Results:
  ✅ 3 passed, ❌ 1 failed, ⏭️ 1 skipped

[See detailed table above]
```

## Integration with Other Commands

Works alongside:
- `/collect-ac` - Manual AC collection (this automates it)
- `/generate-e2e-tests` - Manual test generation (this automates it)
- `/verify-ac` - Manual verification (this automates it)
- `/post-to-jira` - Can use to post results

**This command is a complete automation** of the manual workflow!
