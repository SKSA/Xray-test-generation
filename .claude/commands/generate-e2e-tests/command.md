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

### Step 2: Choose Starting Point

Ask user if they have a JIRA ticket or want to test a component directly:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 Test Generation Starting Point
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

How would you like to start?

1. 🎫 JIRA Ticket
   Generate tests from JIRA ticket (ACs, X-Ray test cases)
   
2. 🧩 Component/Feature
   Generate tests for a specific component or feature
   (No JIRA ticket needed)

Choose option (1 or 2):
```

**Store choice:** `STARTING_POINT = "jira" | "component"`

---

**If user selects "1. JIRA Ticket":**

Ask user: "What's your JIRA ticket number? (e.g., EPS-1234)"

Store as `$TICKET`

Read AC from: `.ac-verification/$TICKET/ac-checklist.md`

**Then proceed to Step 3 (Choose Test Source)** with JIRA-based options.

---

**If user selects "2. Component/Feature":**

Skip JIRA ticket loading and ask for component details:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧩 Component/Feature Details
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Enter component name or path:
Examples:
  - EmailVerification
  - src/components/EmailVerification.tsx
  - Login form
  - User profile page

Component:
```

Store as `$COMPONENT`

**Then ask for test description (free text):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 What Should Be Tested?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Describe what you want to test (free text):

Examples:
  - "Test user can enter email and click send verification code"
  - "Test form validation, submission, and error handling"
  - "Test navigation between login and signup"
  - "Test all user interactions and state changes"

What to test:
```

Store as `$TEST_DESCRIPTION`

**Then search for component in codebase:**

```bash
# Search for component
find src -name "*$COMPONENT*" -type f 2>/dev/null | grep -E '\.(tsx?|jsx?|vue|svelte)$'
```

If multiple matches found:
```
Found multiple matches:
  1. src/components/EmailVerification/EmailVerification.tsx
  2. src/components/EmailVerification/index.tsx
  3. src/pages/Profile/EmailVerification.tsx

Choose component (1-3):
```

If no matches found:
```
⚠️ Component not found in codebase

Options:
  1. Enter a different component name/path
  2. Continue anyway (will generate generic tests)

Choose option (1 or 2):
```

**Read and analyze component:**
- Extract props, functions, state
- Identify user interactions
- Find test IDs/selectors
- Understand component behavior

Display analysis:
```
✅ Component Analysis Complete

Component: EmailVerification.tsx
Location: src/components/EmailVerification/

Props:
  • email: string
  • onVerify: (code: string) => void
  • isLoading: boolean

User Interactions:
  • Input field for verification code
  • Submit button
  • Resend code button

Test IDs Found:
  • verification-code-input
  • submit-verification-button
  • resend-code-button

Test Description:
  "Test user can enter email and click send verification code"

Will generate tests for:
  ✓ Entering verification code (from test description)
  ✓ Clicking submit button (from test description)
  ✓ Resending code (from component analysis)
  ✓ Form validation (from component analysis)
  ✓ Loading states (from component analysis)
```

**Skip to Step 4 (Choose Platform)** - No need for "Choose Test Source" step.

---

### Step 3: Choose Test Source (Only for JIRA Ticket flow)

**This step is SKIPPED if user selected "Component/Feature" in Step 2.**

**Only runs if STARTING_POINT = "jira":**

Ask user what to base the tests on:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Select Test Source (JIRA Ticket: $TICKET)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What should the tests be based on?

1. 📋 Acceptance Criteria (ACs)
   Generate tests from acceptance criteria in ac-checklist.md
   
2. 🧪 X-Ray Test Cases
   Generate tests from X-Ray test cases created earlier
   
3. 🧩 Specific Component (within this ticket)
   Generate tests for a specific component/file

Choose source (1, 2, or 3):
```

**Store choice:** `TEST_SOURCE = "ac" | "xray" | "component"`

**If user selects "1. Acceptance Criteria":**
- Use ACs from `.ac-verification/$TICKET/ac-checklist.md`
- Generate tests based on AC descriptions
- Continue to Step 4

**If user selects "2. X-Ray Test Cases":**
- Check if X-Ray test mapping exists: `.ac-verification/$TICKET/xray-test-mapping.json`
- If exists:
  ```
  ✅ Found X-Ray test cases:
    • EPS-5678 - Verify email verification code request
    • EPS-5679 - Verify verification email delivery time
    • EPS-5680 - Verify code expiration after 15 minutes
    • EPS-5681 - Verify success message after verification
    • EPS-5682 - Verify invalid code error messages
  
  Fetching test case details from JIRA...
  ```
- Fetch full test case details using JIRA CLI:
  ```bash
  for testKey in ${TEST_KEYS[@]}; do
    jira issue view $testKey --plain
  done
  ```
- Extract test steps/scenarios from X-Ray test cases
- Use these as basis for test generation
- Continue to Step 4

- If NOT exists:
  ```
  ⚠️ No X-Ray test cases found for $TICKET
  
  You can:
    1. Create X-Ray test cases first: /create-xray-tests $TICKET
    2. Use acceptance criteria instead (option 1)
    3. Specify a component (option 3)
  
  Which option? (1, 2, or 3):
  ```

**If user selects "3. Specific Component":**
- Ask: "Which component/file do you want to test?"
  ```
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🧩 Select Component to Test
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  
  Enter component path or name:
  Examples:
    - src/components/EmailVerification.tsx
    - EmailVerification
    - src/pages/Profile
  
  Component path:
  ```

- Search for component in codebase:
  ```bash
  # Search for component
  find src -name "*$COMPONENT*" -type f
  ```

- If multiple matches, show list and ask to select:
  ```
  Found multiple matches:
    1. src/components/EmailVerification/EmailVerification.tsx
    2. src/components/EmailVerification/index.tsx
    3. src/pages/Profile/EmailVerification.tsx
  
  Choose component (1-3):
  ```

- Read component file and analyze:
  - Extract props, functions, state
  - Identify user interactions
  - Find test IDs/selectors in component
  - Understand component behavior

- Display component analysis:
  ```
  ✅ Component Analysis Complete
  
  Component: EmailVerification.tsx
  Location: src/components/EmailVerification/
  
  Props:
    • email: string
    • onVerify: (code: string) => void
    • isLoading: boolean
  
  User Interactions:
    • Input field for verification code
    • Submit button
    • Resend code button
    
  Test IDs Found:
    • verification-code-input
    • submit-verification-button
    • resend-code-button
    
  Will generate tests for:
    ✓ Entering verification code
    ✓ Submitting verification
    ✓ Resending code
    ✓ Error handling
    ✓ Loading states
  ```

- Use component analysis as basis for test generation
- Continue to Step 4

### Step 4: Choose Platform First

**Before scanning or detecting frameworks, ask the user what they're testing:**

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

This determines which frameworks to show in the next step.

### Step 5: Smart Selector Scan (Automatic)

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

### Step 6: Scan Repository & Detect Stack

Analyze codebase:

**Frontend Detection:**
```javascript
// Check package.json
const deps = readPackageJson();

// Detect framework
if (deps['react-native']) {
  framework = 'React Native';
  platform = 'mobile';
} else if (deps.react) {
  framework = 'React';
  platform = 'web';
} else if (deps.next) {
  framework = 'Next.js';
  platform = 'web';
} else if (deps.vue) {
  framework = 'Vue';
  platform = 'web';
}

// Detect existing test setup
if (deps.playwright) testFrameworks.push('Playwright');
if (deps.cypress) testFrameworks.push('Cypress');
if (deps.detox) testFrameworks.push('Detox');
if (deps['@wdio/cli']) testFrameworks.push('Maestro');
```

Display findings:

**For Web:**
```
📊 Repository Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Platform: 🌐 Web
Frontend: Next.js 14 (React, TypeScript)
Backend: Node.js/Express API

Installed Test Frameworks:
  ✅ Playwright (E2E)
  ✅ Jest (Unit/Integration)

Test Location: tests/
Naming: *.spec.ts, *.test.ts
Selectors: data-testid attributes (87% coverage)
```

**For Mobile React Native:**
```
📊 Repository Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Platform: 📱 Mobile (React Native)
Version: React Native 0.73
TypeScript: Yes

Installed Test Frameworks:
  ⚠️ No mobile E2E framework detected

Test Location: __tests__/
Naming: *.test.tsx
Selectors: testID props (78% coverage)
Platforms: iOS, Android
```

### Step 7: Choose Test Framework (Based on Platform)

**Now show framework options based on the platform selected in Step 3:**

Ask user: "Which test framework do you want to use?"

**If user selected Web (Platform 1):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌐 Select Web Test Framework
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Detected in your project:
  1. ✅ Playwright (installed)     - Recommended for E2E
  2. ✅ Jest (installed)           - For unit tests
  3. ✅ Supertest (installed)      - For API tests

Not installed (available):
  4. Cypress - Alternative E2E framework
  5. Vitest - Modern unit testing
  6. React Testing Library - Component tests

Choose one or more (comma separated): 
Example: 1,2,3 for Playwright + Jest + Supertest
```

**If user selected Mobile React Native (Platform 2):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📱 Select Mobile Test Framework
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Recommended for React Native:
  1. 🎯 Maestro (recommended)      - Simple YAML-based E2E tests
  2. Detox                        - Built for React Native
  3. Appium                       - Cross-platform standard
  4. Jest                         - For unit tests
  5. React Native Testing Library - Component tests

Detected in your project:
  ✅ Jest (installed)

Choose one or more (comma separated):
Example: 1,5 for Maestro + React Native Testing Library
```

**Smart defaults based on detection:**

For Web:
- If Playwright installed → Default to Playwright
- If Cypress installed → Default to Cypress  
- If neither → Recommend Playwright (offer to install)

For Mobile React Native:
- Always recommend Maestro first (simplest)
- If Detox installed → Show as option
- Suggest React Native Testing Library for component tests

**For each framework choice, confirm:**

**Web example:**
```
You selected: Playwright + Jest + Supertest

Will generate:
  ✅ E2E tests (Playwright) - tests/e2e/
  ✅ Unit tests (Jest) - tests/unit/
  ✅ API tests (Supertest) - tests/api/

Continue? (Y/n)
```

**Mobile React Native example:**
```
You selected: Maestro + React Native Testing Library

Will generate:
  ✅ E2E tests (Maestro) - .maestro/
  ✅ Component tests (React Native Testing Library) - __tests__/

Installation needed:
  - Maestro: curl -Ls "https://get.maestro.mobile.dev" | bash
  - @testing-library/react-native: npm install -D @testing-library/react-native

Continue? (Y/n)
```

### Step 8: Analyze Test Source & Suggest Test Types

**Based on the test source selected in Step 3:**

**If TEST_SOURCE = "ac" (Acceptance Criteria):**

For each AC, analyze and recommend test types:

```javascript
function analyzeAC(ac) {
  const testTypes = [];
  
  // E2E tests for user-facing behavior
  if (ac.match(/user can|user sees|user receives/i)) {
    testTypes.push('E2E');
  }
  
  // API tests for backend behavior
  if (ac.match(/api|endpoint|returns|sends/i)) {
    testTypes.push('API');
  }
  
  // Unit tests for logic/calculations
  if (ac.match(/calculates|validates|expires|expires after/i)) {
    testTypes.push('Unit');
  }
  
  // Component tests for UI components
  if (ac.match(/button|form|modal|dropdown|shows|displays/i)) {
    testTypes.push('Component');
  }
  
  // Visual tests for appearance
  if (ac.match(/color|size|layout|design|style/i)) {
    testTypes.push('Visual');
  }
  
  // Accessibility tests
  if (ac.match(/accessible|screen reader|keyboard|aria/i)) {
    testTypes.push('Accessibility');
  }
  
  // Performance tests
  if (ac.match(/load time|performance|fast|seconds/i)) {
    testTypes.push('Performance');
  }
  
  return testTypes;
}
```

**If TEST_SOURCE = "xray" (X-Ray Test Cases):**

For each X-Ray test case, analyze based on format:

- **If BDD/Gherkin format:**
  - Parse Given/When/Then steps
  - Map to test scenarios
  - Generate E2E tests for user flows
  - Generate unit tests for business logic

- **If Manual Test Steps format:**
  - Parse test steps and expected results
  - Convert manual steps to automated test actions
  - Generate E2E tests for step-by-step flows
  - Generate assertions for expected results

```javascript
function analyzeXRayTest(testCase) {
  const testTypes = [];
  
  if (testCase.format === 'bdd') {
    // Parse Gherkin scenarios
    testTypes.push('E2E'); // Given/When/Then flows
    if (hasBusinessLogic(testCase)) {
      testTypes.push('Unit'); // Logic validation
    }
  } else if (testCase.format === 'manual') {
    // Parse manual test steps
    testTypes.push('E2E'); // Step-by-step automation
    if (hasAPISteps(testCase)) {
      testTypes.push('API'); // API validation
    }
  }
  
  return testTypes;
}
```

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Analyzing X-Ray Test Cases
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Test: EPS-5678 - Verify email verification code request
Format: Manual Test Steps
Recommended: E2E, Component

Test: EPS-5679 - Verify verification email delivery time
Format: Manual Test Steps
Recommended: E2E, API, Performance

Test: EPS-5680 - Verify code expiration after 15 minutes
Format: BDD (Gherkin)
Recommended: E2E, Unit

...
```

**If TEST_SOURCE = "component" (Specific Component):**

Analyze component and generate tests for:

```javascript
function analyzeComponent(component) {
  const testTypes = [];
  
  // Always include component tests
  testTypes.push('Component');
  
  // Check for user interactions → E2E
  if (hasUserInteractions(component)) {
    testTypes.push('E2E');
  }
  
  // Check for business logic → Unit
  if (hasBusinessLogic(component)) {
    testTypes.push('Unit');
  }
  
  // Check for API calls → Integration
  if (hasAPIcalls(component)) {
    testTypes.push('Integration');
  }
  
  // Check for accessibility attributes → A11y
  if (hasA11yAttributes(component)) {
    testTypes.push('Accessibility');
  }
  
  return testTypes;
}
```

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧩 Analyzing Component: EmailVerification.tsx
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Component Type: Form with validation
User Interactions: Input, Submit, Resend
API Calls: POST /api/verify-email
Business Logic: Code validation, Timer countdown

Recommended Test Types:
  ✓ Component tests - Props, rendering, events
  ✓ E2E tests - User flows
  ✓ Integration tests - API integration
  ✓ Unit tests - Validation logic, timer
```
```

Display analysis:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 AC Analysis & Test Recommendations
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC #1: User receives verification email
  Recommended: E2E, API
  Frameworks: Playwright, Supertest

AC #2: Code expires after 15 minutes
  Recommended: Unit, E2E
  Frameworks: Jest, Playwright

AC #3: Success notification shown
  Recommended: Component, E2E
  Frameworks: React Testing Library, Playwright

AC #4: Page loads in under 2 seconds
  Recommended: Performance, E2E
  Frameworks: Lighthouse, Playwright
```

### Step 9: Generate Tests Using Selector Map

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
