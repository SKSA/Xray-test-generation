---
name: /generate-e2e-tests
argument-hint: '[JIRA-TICKET (optional)] [--framework=playwright|cypress|jest|rtl|vitest]'
description: Generate tests from JIRA ticket or component with multi-framework support
mode: single-agent
---

# Generate Tests (Multi-Framework)

## Purpose
Generate automated tests from JIRA tickets (ACs, X-Ray test cases) OR directly from components. Works without JIRA if you just want to test a specific component or feature.

**⚠️ Important:** Run this AFTER implementing your feature, not before. The command scans your code to generate accurate tests.

## Prerequisites

Check dependencies before first use:
```bash
/setup-qa-assistant
→ Checks for Playwright/Cypress/Maestro
→ Offers to install missing frameworks
```

## Workflow

### Step 1: Check Dependencies (Automatic)

Before generating tests, check if required tools are installed:

```bash
# Check for test frameworks based on platform choice
if platform == "web":
  check_playwright_or_cypress()
elif platform == "mobile":
  check_maestro_or_detox()

# If missing, prompt user
if missing_frameworks:
  display_missing_message()
  ask_to_install()
```

**Example prompt:**
```
⚠️ Missing Dependencies Detected

For Web E2E testing, you need one of:
  ❌ Playwright - Not installed
  ❌ Cypress - Not installed

Would you like me to install Playwright? (Y/n)

If yes:
  npm install -D @playwright/test
  npx playwright install
```

### Step 2: What to Generate Tests For

Ask user what they want to generate tests for (accepts JIRA ticket OR component name):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 Generate E2E Tests
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What do you want to generate tests for?

Examples:
  • JIRA ticket: EPS-1234
  • Component: PremiumMealsActiveRewardCard.tsx
  • Component path: src/components/PremiumMealsActiveRewardCard.tsx
  • Feature: Login form validation

Enter ticket or component:
```

**Parse user input:**

```javascript
function parseInput(input) {
  // Check if it's a JIRA ticket (format: ABC-1234)
  if (/^[A-Z]+-\d+$/i.test(input)) {
    return {
      type: 'jira',
      value: input.toUpperCase()
    };
  }
  
  // Otherwise treat as component/feature
  return {
    type: 'component',
    value: input
  };
}
```

**If JIRA ticket detected (e.g., "EPS-1234"):**
- Store as `$TICKET`
- Try to read AC from: `.ac-verification/$TICKET/ac-checklist.md`
- If exists: Show AC summary
- If NOT exists: Offer to run `/collect-ac $TICKET` first, or continue without ACs
- Set `TEST_SOURCE = "jira"`
- Ask if they want to test specific component within ticket or use ACs

**If component/feature detected (e.g., "PremiumMealsActiveRewardCard.tsx"):**
- Store as `$COMPONENT`
- Set `TEST_SOURCE = "component"`
- Search for component in codebase:
  ```bash
  find src -name "*$COMPONENT*" -type f 2>/dev/null | grep -E '\.(tsx?|jsx?|vue|svelte)$'
  ```

Display found component:
```
✅ Found Component

Component: PremiumMealsActiveRewardCard.tsx
Location: src/components/rewards/PremiumMealsActiveRewardCard.tsx

Continue with this component? (Y/n)
```

If multiple matches, show list and ask to select.

**Then ask for test description (free text):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 What Should Be Tested?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Describe what you want to test (optional - press Enter to skip):

Examples:
  - "Test card displays reward information correctly"
  - "Test click interaction opens reward details"
  - "Test loading and error states"
  - Leave blank to auto-detect from component

What to test:
```

Store as `$TEST_DESCRIPTION`

If blank, will auto-analyze component for test scenarios.

### Step 3: Auto-Detect Platform and Frameworks

**Automatically detect platform and available frameworks:**

```bash
# Check package.json
const deps = readPackageJson();

# Detect platform
if (deps['react-native']) {
  detectedPlatform = 'Mobile (React Native)';
} else if (deps.react || deps.next) {
  detectedPlatform = 'Web (React/Next.js)';
}

# Detect existing test frameworks
detectedFrameworks = [];
if (deps.playwright) detectedFrameworks.push('Playwright');
if (deps.cypress) detectedFrameworks.push('Cypress');
if (deps.jest) detectedFrameworks.push('Jest');
if (deps.detox) detectedFrameworks.push('Detox');
if (deps['@testing-library/react']) detectedFrameworks.push('React Testing Library');
```

**Display auto-detected settings:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Auto-Detected Settings
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Platform: 🌐 Web (React/Next.js)

Detected Test Frameworks:
  ✅ Playwright 1.54.0 (E2E tests)
  ✅ Jest 30.2.0 (Unit/Component tests)
  ✅ Cypress 15.4.0 (E2E alternative)
  ✅ React Testing Library (Component tests)

Recommended for this component:
  → Jest + React Testing Library
  (Matches existing test patterns in this feature)
```

### Step 4: Ask User - Auto or Manual? (MANDATORY)

**⚠️ CRITICAL: You MUST ask the user this question. Do NOT skip this step.**

**DO NOT auto-generate without asking the user first.**

**Ask user if they want to use auto-detected settings or choose manually:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚙️ Test Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️  STOP - User choice required

Do you want to use auto-detected settings or choose manually?

1. 🤖 Auto (Recommended)
   Use detected settings: Web + Jest + React Testing Library
   
2. ✋ Manual
   Choose platform and frameworks yourself

Choose option (1 or 2):
```

**WAIT for user response. DO NOT proceed until user answers.**

**If user selects "1. Auto (Recommended)":**
- Use auto-detected platform
- Use auto-detected/recommended frameworks
- Skip to Step 7 (Generate Tests)
- Display confirmation:
  ```
  ✅ Using auto-detected settings:
     Platform: Web
     Frameworks: Jest + React Testing Library
  
  Generating tests...
  ```

**If user selects "2. Manual":**
- Proceed to Step 5 (Manual Platform Selection)
- Then Step 6 (Manual Framework Selection)

### Step 5: Manual Platform Selection (Only if user chose Manual)

**This step is SKIPPED if user chose "Auto" in Step 4.**

**Ask for platform:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Select Testing Platform
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What platform are you testing?

1. 🌐 Web (Browser-based applications)
   Frameworks: Playwright, Cypress, Jest, Vitest
   
2. 📱 Mobile (React Native)
   Frameworks: Maestro, Detox, Appium

Choose platform (1 or 2):
```

**Store platform choice:** `PLATFORM = "web" or "mobile"`

### Step 6: Manual Framework Selection (Only if user chose Manual)

**This step is SKIPPED if user chose "Auto" in Step 4.**

**Based on platform selected in Step 5, show framework options:**

**If Web (Platform 1):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌐 Select Web E2E Framework
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Which technology do you want to use for E2E test generation?

Detected in your project:
  1. ✅ Playwright (installed)     - Recommended for E2E ⭐
  2. ✅ Jest (installed)           - For unit tests
  3. ✅ React Testing Library (installed) - Component tests
  4. ✅ Cypress (installed)        - E2E alternative

Not installed (available):
  5. Vitest - Modern unit testing
  6. Supertest - API tests

Choose framework (comma separated for multiple):
Example: 1,2 for Playwright + Jest
```

**If Mobile (Platform 2):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📱 Select Mobile E2E Framework
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Which technology do you want to use for E2E test generation?

Recommended for React Native:
  1. 🎯 Maestro (recommended)      - Simple YAML-based E2E tests ⭐
  2. Detox                        - Built for React Native
  3. Appium                       - Cross-platform standard
  4. Jest                         - For unit tests
  5. React Native Testing Library - Component tests

Detected in your project:
  ✅ Jest (installed)

Choose framework (comma separated for multiple):
Example: 1,5 for Maestro + React Native Testing Library
```

**Confirm selection:**
```
You selected: Playwright + Jest

Will generate:
  ✅ E2E tests (Playwright) - tests/e2e/BoxDiscountActiveRewardCard.spec.ts
  ✅ Unit tests (Jest) - tests/unit/BoxDiscountActiveRewardCard.test.ts

Continue? (Y/n)
```

### Step 7: Smart Selector Scan (Automatic)

**Automatically scans YOUR IMPLEMENTED CODE for selectors:**

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Scanning Your Code for Selectors...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute smart selector scan on your implementation:

**For Web (Platform 1):**
```bash
# Scans your actual code
→ Scans src/ for data-testid, data-cy, aria-label
→ Finds selectors YOU added to your components
→ Builds selector map from YOUR code
→ Detects patterns and naming conventions
→ Identifies components you implemented
```

**For Mobile React Native (Platform 2):**
```bash
# Scans your React Native code
→ Scans src/ for testID props
→ Finds testID YOU added to your RN components
→ Builds selector map from YOUR code
→ Detects React Native component patterns
→ Identifies screens and navigators
```

Show results:

**For Web:**
```
✅ Selector Scan Complete

Found: 342 selectors in your code
  • Buttons: 45
  • Inputs: 78
  • Forms: 23
  • Pages: 12

Quality: 87% coverage
Missing: 52 elements need selectors in your implementation

Selector map saved: .ac-verification/selectors.json
```

**For Mobile React Native:**
```
✅ Selector Scan Complete

Found: 156 testIDs in your React Native code
  • Buttons: 28
  • TextInputs: 34
  • Touchables: 42
  • Screens: 8

Quality: 78% coverage
Missing: 38 elements need testID props

Selector map saved: .ac-verification/selectors.json
```

If quality is low (<70%), warn:

**For Web:**
```
⚠️ Selector coverage is low (52%)
Your implementation is missing test selectors.
Consider adding data-testid attributes to key elements.
```

**For Mobile React Native:**
```
⚠️ testID coverage is low (52%)
Your React Native components are missing testID props.
Consider adding testID to key components:
<Button testID="submit-button" />
```

### Step 8: Generate Tests

Based on what was entered in Step 2:

**If Component (e.g., "PremiumMealsActiveRewardCard.tsx"):**
- Use component analysis + test description (if provided)
- Use selectors found in Step 5
- Generate tests for identified interactions

**If JIRA Ticket (e.g., "EPS-1234"):**
- Use ACs or X-Ray test cases
- Use selectors found in Step 5
- Generate tests covering all acceptance criteria

**Generate test files with selected frameworks from Step 4:**

Continue with test generation using the selector map and chosen frameworks...

**Automatically scans YOUR IMPLEMENTED CODE for selectors:**

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Scanning Your Code for Selectors...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute smart selector scan on your implementation:

**For Web (Platform 1):**
```bash
# Scans your actual code
→ Scans src/ for data-testid, data-cy, aria-label
→ Finds selectors YOU added to your components
→ Builds selector map from YOUR code
→ Detects patterns and naming conventions
→ Identifies components you implemented
```

**For Mobile React Native (Platform 2):**
```bash
# Scans your React Native code
→ Scans src/ for testID props
→ Finds testID YOU added to your RN components
→ Builds selector map from YOUR code
→ Detects React Native component patterns
→ Identifies screens and navigators
```

Show results:

**For Web:**
```
✅ Selector Scan Complete

Found: 342 selectors in your code
  • Buttons: 45
  • Inputs: 78
  • Forms: 23
  • Pages: 12

Quality: 87% coverage
Missing: 52 elements need selectors in your implementation

Selector map saved: .ac-verification/selectors.json
```

**For Mobile React Native:**
```
✅ Selector Scan Complete

Found: 156 testIDs in your React Native code
  • Buttons: 28
  • TextInputs: 34
  • Touchables: 42
  • Screens: 8

Quality: 78% coverage
Missing: 38 elements need testID props

Selector map saved: .ac-verification/selectors.json
```

If quality is low (<70%), warn:

**For Web:**
```
⚠️ Selector coverage is low (52%)
Your implementation is missing test selectors.
Consider adding data-testid attributes to key elements.
```

**For Mobile React Native:**
```
⚠️ testID coverage is low (52%)
Your React Native components are missing testID props.
Consider adding testID to key components:
<Button testID="submit-button" />
```

### Step 8: Generate Tests

Based on what was entered in Step 2 and settings from Steps 3-6:

**If Component (e.g., "BoxDiscountActiveRewardCard.tsx"):**
- Use component analysis + test description (if provided)
- Use selectors found in Step 7
- Use frameworks chosen in Step 4 or Step 6
- Generate tests for identified interactions

**If JIRA Ticket (e.g., "BDPI-1234"):**
- Use ACs or X-Ray test cases
- Use selectors found in Step 7
- Use frameworks chosen in Step 4 or Step 6
- Generate tests covering all acceptance criteria

**Generate test files:**

Display progress:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 Generating Tests
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Component: BoxDiscountActiveRewardCard.tsx
Frameworks: Jest + React Testing Library

Creating test file...
  ✅ BoxDiscountActiveRewardCard.spec.tsx (222 lines)

Test scenarios:
  ✓ Title renders with formatted fixed budget
  ✓ Title renders with formatted percent budget
  ✓ CTA shows "Applied"
  ✓ Click handler is called
  ✓ Expiration label renders
  ✓ Props forwarded correctly
```

Continue with test execution and verification...

### Step 9: Display Summary and Next Steps

**Use the selector map from Step 2:**

```typescript
// Load selector map
const selectorMap = JSON.parse(
  fs.readFileSync('.ac-verification/selectors.json', 'utf8')
);

// Find real selectors for components
function getSelector(componentName) {
  // Check if selector exists in map
  const selector = selectorMap.selectors.buttons[componentName] ||
                   selectorMap.selectors.inputs[componentName];
  
  if (selector) {
    return `[${selector.type}="${selector.value}"]`;
  }
  
  // Fallback to generic
  console.warn(`No selector found for ${componentName}, using generic`);
  return `[data-testid="${componentName}"]`;
}
```

### Step 10: Generate Tests for Selected Frameworks

#### A. Playwright (E2E)

```typescript
import { test, expect } from '@playwright/test';

test.describe('EPS-1234: Email Verification', () => {
  
  /**
   * AC #1: User receives verification email within 30 seconds
   * 
   * Given user is on signup page
   * When user submits valid email
   * Then verification email is sent within 30 seconds
   */
  test('AC #1: User receives verification email', async ({ page }) => {
    // Given - user is on signup page
    await page.goto('/signup');
    
    // When - user submits valid email
    // Using REAL selectors from selector map:
    const testEmail = `test-${Date.now()}@example.com`;
    await page.fill('[data-testid="email-input"]', testEmail); // ← From selector scan
    await page.click('[data-testid="submit-button"]'); // ← From selector scan
    
    // Then - verification email sent confirmation shown
    await expect(page.locator('[data-testid="success-message"]'))
      .toHaveText(/email sent/i, { timeout: 30000 });
    
    // Verify email was actually sent (API check)
    const response = await page.request.get(
      `/api/test/emails?to=${testEmail}`
    );
    expect(response.status()).toBe(200);
    const emails = await response.json();
    expect(emails).toHaveLength(1);
    expect(emails[0].subject).toContain('Verify');
  });

  /**
   * AC #2: Code expires after 15 minutes
   */
  test('AC #2: Code expires after 15 minutes', async ({ page }) => {
    // This would be a long-running test - consider time mocking
    // Using time manipulation with Playwright
    await page.goto('/verify');
    
    // Fast-forward time by 16 minutes
    await page.addInitScript(() => {
      const now = Date.now();
      Date.now = () => now + (16 * 60 * 1000);
    });
    
    // Attempt to verify with expired code
    await page.fill('[data-testid="code-input"]', '123456');
    await page.click('[data-testid="verify-button"]');
    
    // Should show expiration error
    await expect(page.locator('[data-testid="error-message"]'))
      .toHaveText(/expired/i);
  });
  
  /**
   * AC #3: Success notification shown
   */
  test('AC #3: Success notification displayed', async ({ page }) => {
    await page.goto('/verify');
    
    // Enter valid code
    await page.fill('[data-testid="code-input"]', '123456');
    await page.click('[data-testid="verify-button"]');
    
    // Success notification appears
    const notification = page.locator('[data-testid="notification"]');
    await expect(notification).toBeVisible();
    await expect(notification).toHaveClass(/success/);
    await expect(notification).toHaveText(/verified successfully/i);
    
    // Notification auto-dismisses after 3s
    await expect(notification).not.toBeVisible({ timeout: 4000 });
  });
});
```

#### B. Jest + Supertest (API Tests)

```typescript
import request from 'supertest';
import app from '../src/app';
import { setupTestDB, teardownTestDB } from './helpers/db';

describe('EPS-1234: Email Verification API', () => {
  beforeAll(async () => {
    await setupTestDB();
  });

  afterAll(async () => {
    await teardownTestDB();
  });

  /**
   * AC #1: User receives verification email
   * API Test: POST /api/auth/send-verification
   */
  test('AC #1 (API): Send verification email', async () => {
    const response = await request(app)
      .post('/api/auth/send-verification')
      .send({
        email: 'test@example.com'
      })
      .expect(200);

    expect(response.body).toMatchObject({
      success: true,
      message: expect.stringContaining('sent'),
      emailId: expect.any(String)
    });

    // Verify email job was queued
    // (assuming you use a queue like Bull)
    const jobs = await emailQueue.getJobs();
    expect(jobs).toHaveLength(1);
    expect(jobs[0].data.to).toBe('test@example.com');
  });

  /**
   * AC #2: Code expires after 15 minutes
   * API Test: POST /api/auth/verify with expired code
   */
  test('AC #2 (API): Expired code returns 401', async () => {
    // Create an expired verification code
    const expiredCode = await createExpiredVerificationCode({
      email: 'test@example.com',
      expiresAt: new Date(Date.now() - 1000) // 1 second ago
    });

    const response = await request(app)
      .post('/api/auth/verify')
      .send({
        email: 'test@example.com',
        code: expiredCode.code
      })
      .expect(401);

    expect(response.body).toMatchObject({
      error: 'Code expired',
      code: 'VERIFICATION_CODE_EXPIRED'
    });
  });
});
```

#### C. Jest + React Testing Library (Component Tests)

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { VerificationForm } from './VerificationForm';
import { mockEmailService } from '../mocks/emailService';

describe('EPS-1234: Verification Form Component', () => {
  
  /**
   * AC #1: User receives verification email
   * Component Test: Form submission triggers email
   */
  test('AC #1 (Component): Form triggers email send', async () => {
    const user = userEvent.setup();
    const onSuccess = jest.fn();
    
    render(<VerificationForm onSuccess={onSuccess} />);
    
    // User enters email
    const emailInput = screen.getByLabelText(/email/i);
    await user.type(emailInput, 'test@example.com');
    
    // User submits
    const submitButton = screen.getByRole('button', { name: /send/i });
    await user.click(submitButton);
    
    // Loading state shown
    expect(submitButton).toBeDisabled();
    expect(screen.getByText(/sending/i)).toBeInTheDocument();
    
    // Success message appears
    await waitFor(() => {
      expect(screen.getByText(/email sent/i)).toBeInTheDocument();
    });
    
    // Email service was called
    expect(mockEmailService.sendVerification).toHaveBeenCalledWith({
      email: 'test@example.com'
    });
  });

  /**
   * AC #3: Success notification shown
   */
  test('AC #3 (Component): Success notification displays', async () => {
    render(<VerificationForm />);
    
    // Trigger success (mock API response)
    mockEmailService.sendVerification.mockResolvedValue({ success: true });
    
    const user = userEvent.setup();
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.click(screen.getByRole('button', { name: /send/i }));
    
    // Notification appears
    const notification = await screen.findByRole('alert');
    expect(notification).toHaveClass('notification-success');
    expect(notification).toHaveTextContent(/verified successfully/i);
    
    // Notification auto-dismisses
    await waitFor(() => {
      expect(notification).not.toBeInTheDocument();
    }, { timeout: 4000 });
  });
});
```

#### D. Cypress (Alternative E2E)

```javascript
describe('EPS-1234: Email Verification', () => {
  
  /**
   * AC #1: User receives verification email
   */
  it('AC #1: User receives verification email', () => {
    cy.visit('/signup');
    
    // Intercept email API
    cy.intercept('POST', '/api/auth/send-verification', {
      statusCode: 200,
      body: { success: true }
    }).as('sendEmail');
    
    // User submits email
    cy.get('[data-cy=email-input]').type('test@example.com');
    cy.get('[data-cy=submit-button]').click();
    
    // Wait for API call
    cy.wait('@sendEmail');
    
    // Success message shown
    cy.get('[data-cy=success-message]')
      .should('be.visible')
      .and('contain', 'email sent');
  });
  
  /**
   * AC #2: Code expires after 15 minutes
   */
  it('AC #2: Expired code shows error', () => {
    // Use Cypress clock to fast-forward time
    cy.clock();
    
    cy.visit('/verify');
    cy.get('[data-cy=code-input]').type('123456');
    
    // Fast-forward 16 minutes
    cy.tick(16 * 60 * 1000);
    
    cy.get('[data-cy=verify-button]').click();
    
    // Error shown
    cy.get('[data-cy=error-message]')
      .should('contain', 'expired');
  });
});
```

#### E. Lighthouse (Performance)

```javascript
import lighthouse from 'lighthouse';
import * as chromeLauncher from 'chrome-launcher';

/**
 * AC #4: Page loads in under 2 seconds
 */
test('AC #4 (Performance): Page loads under 2s', async () => {
  const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });
  
  const options = {
    logLevel: 'info',
    output: 'json',
    port: chrome.port
  };
  
  const runnerResult = await lighthouse('http://localhost:3000/signup', options);
  
  // Performance metrics
  const metrics = runnerResult.lhr.audits['metrics'].details.items[0];
  
  // Time to Interactive under 2 seconds
  expect(metrics.interactive).toBeLessThan(2000);
  
  // First Contentful Paint under 1 second
  expect(metrics.firstContentfulPaint).toBeLessThan(1000);
  
  // Performance score above 90
  expect(runnerResult.lhr.categories.performance.score).toBeGreaterThan(0.9);
  
  await chrome.kill();
});
```

#### F. Axe (Accessibility)

```typescript
import { test, expect } from '@playwright/test';
import { injectAxe, checkA11y, getViolations } from 'axe-playwright';

/**
 * AC #5: Accessible to screen readers
 */
test('AC #5 (Accessibility): Page is accessible', async ({ page }) => {
  await page.goto('/signup');
  await injectAxe(page);
  
  // Run accessibility checks
  await checkA11y(page, null, {
    detailedReport: true,
    detailedReportOptions: {
      html: true
    }
  });
  
  // Get violations
  const violations = await getViolations(page);
  
  // Ensure no critical violations
  const criticalViolations = violations.filter(v => 
    v.impact === 'critical' || v.impact === 'serious'
  );
  
  expect(criticalViolations).toHaveLength(0);
  
  // Check specific WCAG criteria
  expect(violations.find(v => v.id === 'color-contrast')).toBeUndefined();
  expect(violations.find(v => v.id === 'label')).toBeUndefined();
});
```

**Add selector source comments:**
```typescript
test('AC #1: Email input validation', async ({ page }) => {
  // Selector: email-input (found in src/components/Form.tsx:45)
  // Quality: ✅ High (data-testid)
  await page.fill('[data-testid="email-input"]', 'test@example.com');
  
  // Selector: submit-button (found in src/components/Form.tsx:52)
  // Quality: ✅ High (data-testid)
  await page.click('[data-testid="submit-button"]');
});
```

**B. For Mobile React Native - Maestro E2E Tests**

Generate `.maestro/EPS-1234.yaml`:

```yaml
# AC Verification Tests for EPS-1234
# Generated from acceptance criteria
appId: com.yourapp

---
# AC #1: User can login with valid credentials
- launchApp
- tapOn:
    id: "email-input"  # From selector scan: src/screens/LoginScreen.tsx:34
- inputText: "test@example.com"
- tapOn:
    id: "password-input"  # From selector scan: src/screens/LoginScreen.tsx:42
- inputText: "password123"
- tapOn:
    id: "login-button"  # From selector scan: src/screens/LoginScreen.tsx:50
- assertVisible: "Welcome!"

---
# AC #2: User sees error with invalid email
- launchApp
- tapOn:
    id: "email-input"
- inputText: "invalid-email"
- tapOn:
    id: "login-button"
- assertVisible:
    id: "error-message"
    text: "Invalid email format"
```

**C. For Mobile React Native - Detox Tests (Alternative)**

Generate `e2e/EPS-1234.e2e.js`:

```javascript
// AC Verification Tests for EPS-1234
describe('Login flow - EPS-1234', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  // AC #1: User can login with valid credentials
  it('should login successfully with valid credentials', async () => {
    // Selector: email-input (found in src/screens/LoginScreen.tsx:34)
    await element(by.id('email-input')).typeText('test@example.com');
    
    // Selector: password-input (found in src/screens/LoginScreen.tsx:42)
    await element(by.id('password-input')).typeText('password123');
    
    // Selector: login-button (found in src/screens/LoginScreen.tsx:50)
    await element(by.id('login-button')).tap();
    
    // Assert success
    await expect(element(by.text('Welcome!'))).toBeVisible();
  });

  // AC #2: User sees error with invalid email
  it('should show error for invalid email', async () => {
    await element(by.id('email-input')).typeText('invalid-email');
    await element(by.id('login-button')).tap();
    
    await expect(element(by.id('error-message'))).toBeVisible();
    await expect(element(by.text('Invalid email format'))).toBeVisible();
  });
});
```

**D. For Mobile React Native - Component Tests**

Generate `__tests__/LoginScreen.test.tsx`:

```typescript
// Component tests for EPS-1234
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import LoginScreen from '../src/screens/LoginScreen';

describe('LoginScreen - EPS-1234', () => {
  // AC #1: User can enter credentials
  it('should allow entering email and password', () => {
    const { getByTestId } = render(<LoginScreen />);
    
    const emailInput = getByTestId('email-input');
    const passwordInput = getByTestId('password-input');
    
    fireEvent.changeText(emailInput, 'test@example.com');
    fireEvent.changeText(passwordInput, 'password123');
    
    expect(emailInput.props.value).toBe('test@example.com');
    expect(passwordInput.props.value).toBe('password123');
  });

  // AC #2: Login button is disabled when form is invalid
  it('should disable login button when form is invalid', () => {
    const { getByTestId } = render(<LoginScreen />);
    
    const loginButton = getByTestId('login-button');
    expect(loginButton.props.accessibilityState.disabled).toBe(true);
  });
});
```

### Step 11: Create Test Configuration Files

#### playwright.config.ts
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  timeout: 30000,
  expect: { timeout: 5000 },
  
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Step 12: Create Test Documentation

Create: `tests/e2e/EPS-1234-README.md`

```markdown
# Test Suite: EPS-1234 - Email Verification

## Overview
Automated tests covering all acceptance criteria for email verification feature.

## Test Coverage

| AC # | Description | E2E | API | Component | Unit | Visual | A11y | Perf |
|------|-------------|-----|-----|-----------|------|--------|------|------|
| 1    | User receives email | ✅ | ✅ | ✅ | - | - | - | - |
| 2    | Code expires | ✅ | ✅ | - | ✅ | - | - | - |
| 3    | Notification shown | ✅ | - | ✅ | - | ✅ | - | - |
| 4    | Loads under 2s | - | - | - | - | - | - | ✅ |
| 5    | Accessible | ✅ | - | - | - | - | ✅ | - |

**Total Tests:** 12
- E2E (Playwright): 5 tests
- API (Supertest): 3 tests
- Component (RTL): 2 tests
- Unit (Jest): 1 test
- Performance (Lighthouse): 1 test

## Running Tests

### All Tests
\`\`\`bash
npm test
\`\`\`

### E2E Tests Only
\`\`\`bash
npx playwright test tests/e2e/EPS-1234.spec.ts
\`\`\`

### API Tests Only
\`\`\`bash
npm test -- tests/api/EPS-1234.test.ts
\`\`\`

### Component Tests Only
\`\`\`bash
npm test -- tests/components/VerificationForm.test.tsx
\`\`\`

### Performance Tests
\`\`\`bash
npm run test:performance
\`\`\`

### Accessibility Tests
\`\`\`bash
npm run test:a11y
\`\`\`

## Test Reports

- Playwright: `playwright-report/index.html`
- Jest: `coverage/lcov-report/index.html`
- Lighthouse: `lighthouse-report.html`
- Axe: `a11y-report.html`
```

### Step 13: Display Summary

**For Web:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎭 Tests Generated for EPS-1234 (Web)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Selector Scan:
- Scanned: 156 files
- Found: 342 selectors
- Quality: 87% coverage
- Map: .ac-verification/selectors.json

✅ Test Files Created:
- tests/e2e/EPS-1234.spec.ts (5 tests) - Playwright
- tests/api/EPS-1234.test.ts (3 tests) - Supertest
- tests/unit/expiration.test.ts (1 test) - Jest

✅ Configuration:
- playwright.config.ts
- jest.config.js

✅ Documentation:
- tests/e2e/EPS-1234-README.md

📊 Test Summary:
Total: 9 tests across 3 frameworks
- E2E: 5 tests (Playwright) - Using real selectors ✅
- API: 3 tests (Supertest)
- Unit: 1 test (Jest)

🎯 AC Coverage: 100% (5/5 ACs covered)
🎯 Selector Quality: 87% (uses real selectors, not generic)

🚀 Next Steps:
1. Review generated tests (check selector accuracy)
2. Run tests: npx playwright test
3. If selectors missing, add data-testid attributes
4. Implement feature to pass tests
5. Run: /verify-ac EPS-1234
```

**For Mobile React Native:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📱 Tests Generated for EPS-1234 (React Native)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ testID Scan:
- Scanned: 89 files
- Found: 156 testIDs
- Quality: 78% coverage
- Map: .ac-verification/selectors.json

✅ Test Files Created:
- .maestro/EPS-1234.yaml (5 flows) - Maestro E2E
- __tests__/LoginScreen.test.tsx (4 tests) - Component
- __tests__/validation.test.ts (2 tests) - Unit

✅ Configuration:
- .maestro/.config.yaml
- jest.config.js (React Native preset)

✅ Documentation:
- .maestro/EPS-1234-README.md

📊 Test Summary:
Total: 11 tests across 3 types
- E2E: 5 flows (Maestro) - Using real testIDs ✅
- Component: 4 tests (React Native Testing Library)
- Unit: 2 tests (Jest)

🎯 AC Coverage: 100% (5/5 ACs covered)
🎯 testID Quality: 78% (uses real testIDs from your components)

🚀 Next Steps:
1. Install Maestro: curl -Ls "https://get.maestro.mobile.dev" | bash
2. Review generated tests
3. Run E2E: maestro test .maestro/EPS-1234.yaml
4. Run component tests: npm test
5. If testIDs missing, add to your components:
   <Button testID="login-button" />
6. Run: /verify-ac EPS-1234
```

## Output Files

- `tests/e2e/$TICKET.spec.ts` - Playwright E2E tests
- `tests/api/$TICKET.test.ts` - API tests
- `tests/components/*.test.tsx` - Component tests
- `tests/unit/*.test.ts` - Unit tests
- `tests/performance/$TICKET.perf.ts` - Performance tests
- `tests/e2e/$TICKET-README.md` - Documentation
- Test config files as needed
