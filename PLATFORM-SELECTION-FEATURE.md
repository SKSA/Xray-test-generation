# Platform Selection Feature - Added to /generate-e2e-tests

## ✅ What Changed

The `/generate-e2e-tests` command now supports **both Web and Mobile React Native** platforms!

---

## 🎯 New Workflow

### **Step 1: Platform Selection** (NEW)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Select Platform
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 🌐 Web (Browser-based)
2. 📱 Mobile (React Native)

Choose platform (1 or 2):
```

---

## 🌐 For Web Platform

### **Framework Options:**
```
Detected in your project:
  1. ✅ Playwright (installed)     - Recommended for E2E
  2. ✅ Jest (installed)           - For unit tests
  3. ✅ Supertest (installed)      - For API tests

Not installed (available):
  4. Cypress - Alternative E2E
  5. Vitest - Modern unit testing
  6. React Testing Library - Component tests
```

### **Selector Scanning:**
- Scans for: `data-testid`, `data-cy`, `aria-label`
- Generates: Playwright/Cypress E2E tests

### **Generated Tests:**
```typescript
// tests/e2e/EPS-1234.spec.ts
test('AC #1: Login', async ({ page }) => {
  await page.fill('[data-testid="email-input"]', 'test@example.com');
  await page.click('[data-testid="submit-button"]');
});
```

---

## 📱 For Mobile React Native Platform

### **Framework Options:**
```
Recommended for React Native:
  1. 🎯 Maestro (recommended)      - Simple YAML-based tests
  2. Detox                        - Built for React Native
  3. Appium                       - Cross-platform standard
  4. React Native Testing Library - Component tests

Detected in your project:
  ✅ Jest (installed) - For unit tests
```

### **Selector Scanning:**
- Scans for: `testID` props
- Generates: Maestro YAML / Detox / Component tests

### **Generated Tests (Maestro):**
```yaml
# .maestro/EPS-1234.yaml
appId: com.yourapp
---
- launchApp
- tapOn:
    id: "email-input"
- inputText: "test@example.com"
- tapOn:
    id: "login-button"
- assertVisible: "Welcome!"
```

### **Generated Tests (React Native Testing Library):**
```typescript
// __tests__/LoginScreen.test.tsx
import { render, fireEvent } from '@testing-library/react-native';

test('should login', () => {
  const { getByTestId } = render(<LoginScreen />);
  fireEvent.changeText(getByTestId('email-input'), 'test@example.com');
  fireEvent.press(getByTestId('login-button'));
});
```

---

## 🔄 Complete Workflow

### **For Web:**
```bash
/collect-ac EPS-1234
# Implement web feature
/generate-e2e-tests EPS-1234
→ Select platform: 1 (Web)
→ Choose framework: 1 (Playwright)
→ Generates Playwright tests
npx playwright test
/verify-ac EPS-1234
```

### **For React Native:**
```bash
/collect-ac EPS-1234
# Implement React Native feature
/generate-e2e-tests EPS-1234
→ Select platform: 2 (Mobile React Native)
→ Choose framework: 1 (Maestro)
→ Generates Maestro YAML tests
maestro test .maestro/EPS-1234.yaml
/verify-ac EPS-1234
```

---

## 📊 Comparison

| Feature | Web | Mobile React Native |
|---------|-----|---------------------|
| **Selectors** | data-testid, data-cy | testID props |
| **Primary Framework** | Playwright | Maestro (recommended) |
| **Alternative** | Cypress | Detox |
| **Component Tests** | React Testing Library | React Native Testing Library |
| **Test Location** | tests/e2e/ | .maestro/ or e2e/ |
| **Run Command** | npx playwright test | maestro test |

---

## 🎯 Key Benefits

### **For Web Teams:**
- ✅ Playwright/Cypress E2E tests
- ✅ Jest unit tests
- ✅ Supertest API tests
- ✅ data-testid scanning

### **For React Native Teams:**
- ✅ Maestro (simplest option)
- ✅ Detox (RN-optimized)
- ✅ React Native Testing Library
- ✅ testID prop scanning

### **For Full-Stack Teams:**
- ✅ One workflow for both platforms
- ✅ Same commands, different outputs
- ✅ Platform-aware selector scanning
- ✅ Unified AC verification

---

## 🚀 What Gets Generated

### **Web Output:**
```
tests/
├── e2e/
│   └── EPS-1234.spec.ts        (Playwright)
├── api/
│   └── EPS-1234.test.ts        (Supertest)
└── unit/
    └── validation.test.ts      (Jest)
```

### **Mobile React Native Output:**
```
.maestro/
├── EPS-1234.yaml               (Maestro flows)
└── EPS-1234-README.md

__tests__/
├── LoginScreen.test.tsx        (Component tests)
└── validation.test.ts          (Unit tests)
```

---

## 💡 Installation Requirements

### **Web (No extra install needed if you have):**
- Playwright: `npm install -D @playwright/test`
- Cypress: `npm install -D cypress`

### **Mobile React Native:**
- Maestro: `curl -Ls "https://get.maestro.mobile.dev" | bash`
- Detox: `npm install -D detox`
- React Native Testing Library: `npm install -D @testing-library/react-native`

---

## 📝 Updated Documentation

See `/generate-e2e-tests` command for full details on:
- Platform selection
- Framework recommendations
- Selector scanning (per platform)
- Test generation examples
- Configuration files

---

**Committed:** March 22, 2026
**Status:** ✅ Ready to use for both Web and React Native!
