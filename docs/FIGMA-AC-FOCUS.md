# Figma AC Extractor - Focus on User Flows

## ✅ Updated Behavior

The `/figma-ac-extractor` command has been updated to focus on **functional user flows and interactions** instead of visual styling.

---

## 🎯 What It NOW Extracts

### ✅ DOES Extract (Functional/Behavioral):

**1. User Interactions**
- Clicks, taps, swipes
- Hover states
- Drag actions
```
AC-I1: When user clicks Submit button, navigate to Verification screen
AC-I2: When user hovers Submit button, show hover state
```

**2. Navigation & User Flows**
- Screen transitions
- Critical user paths
- Multi-step journeys
```
AC-P1: User flow: Landing → Email Input → Verification → Success
AC-P2: User flow: Landing → Email Input → Error → Retry
```

**3. Form Behaviors**
- Validation rules
- Required fields
- Submission states
- Error handling
```
AC-F1: Email input is required and must show error if empty
AC-F2: Email input must validate email format
AC-F3: Submit button must be disabled until form is valid
```

**4. State Changes**
- Enabled/disabled states
- Loading states
- Error states
- Success states
```
AC-I3: When form is submitting, button shows loading state
AC-I4: When error occurs, show error message
```

**5. Conditional Logic**
- Show/hide behaviors
- Permission-based visibility
- Dynamic content
```
AC-F5: Error message must clear when user starts typing
AC-D1: Show success state for 3 seconds before redirect
```

**6. Functional Accessibility**
- Keyboard navigation
- Focus management
- Screen reader labels
- Error announcements
```
AC-A1: Submit button must be keyboard accessible (Tab, Enter/Space)
AC-A3: Error messages must be announced to screen readers
```

---

## ❌ Does NOT Extract (Visual/Styling):

**Excluded (Designer/Frontend Handles):**
- ❌ Colors (hex codes, RGB values)
- ❌ Font sizes, font families
- ❌ Exact dimensions (320px × 48px)
- ❌ Spacing, padding, margins
- ❌ Border radius, shadows
- ❌ Visual-only accessibility (contrast ratios, touch target sizes)

**Rationale:** These are design implementation details, not functional requirements for QA testing.

---

## 📊 Example Output

**Old Output (Too Visual):**
```
AC-V1: Email input must be 320px × 48px
AC-V4: Submit button background must be #00A862
AC-V7: Heading uses Inter Bold 24px
AC-V10: Form fields have 16px vertical spacing
```

**New Output (Functional):**
```
AC-P1: User flow: Landing → Email Input → Verification → Success
AC-I1: When user clicks Submit, navigate to Verification screen
AC-F1: Email input is required and must show error if empty
AC-F3: Submit button must be disabled until email is valid
AC-A1: Submit button must be keyboard accessible
```

---

## 🛠️ Technical Implementation

### Filter Logic

The extractor now:

1. **Skips visual properties:**
```typescript
// Skip these extractions:
- component.width, component.height
- component.fills (colors)
- component.fontSize, component.fontFamily
- component.paddingLeft, component.margin
- component.borderRadius, component.shadows
```

2. **Focuses on interactions:**
```typescript
// Only extract components with interactions
if (child.interactions && child.interactions.length > 0) {
  extractInteractionAC(child);
}
```

3. **Parses designer notes for behavior:**
```typescript
// Filter out visual-only notes
const visualKeywords = ['color', 'font', 'size', 'px', 'spacing'];
if (!visualKeywords.some(k => note.includes(k))) {
  extractAC(note);
}
```

---

## 🎯 Use Cases

### ✅ Good Use Cases:
- Multi-step user flows (signup, checkout, onboarding)
- Form validation and submission
- Navigation patterns
- State management (loading, error, success)
- Critical user paths
- Interactive prototypes

### ⚠️ Not Ideal For:
- Static marketing pages
- Pure visual designs (no interactions)
- Style guides
- Design systems (colors/typography only)

---

## 🚀 Workflow Integration

```bash
# Step 1: Get functional ACs from JIRA
/collect-ac EPS-1234
→ Fetches business requirements

# Step 2: Add user flow ACs from Figma
/figma-ac-extractor https://figma.com/design/xyz123
→ Adds interaction and navigation ACs
→ Skips visual styling

# Step 3: Generate E2E tests
/generate-e2e-tests EPS-1234
→ Tests user flows, interactions, validations
→ NOT testing colors or font sizes

# Step 4: Verify
/verify-ac EPS-1234
→ Test all functional behaviors
```

---

## 💡 Benefits

**Before (Visual Focus):**
- 23 ACs: 12 visual, 4 interaction, 5 a11y, 2 notes
- Developer wastes time on pixel-perfect validation
- QA tests colors instead of user flows
- Visual details change frequently

**After (Behavioral Focus):**
- 19 ACs: 2 paths, 6 interactions, 5 forms, 4 a11y, 2 notes
- Developer focuses on functionality
- QA tests actual user journeys
- Behavioral ACs stay stable

---

## 📋 Summary

**What Changed:**
✅ Extract user interactions and flows
✅ Extract form behaviors and validation
✅ Extract critical paths
✅ Extract functional accessibility
❌ Skip colors, fonts, dimensions, spacing

**Why:**
- Functional ACs = testable user requirements
- Visual ACs = implementation details for designers/frontend
- Focus on "what users do" not "how it looks"

**Result:**
- Cleaner, more focused acceptance criteria
- Better E2E test generation
- Less noise, more signal
- Tests stay relevant longer

---

**Updated:** March 22, 2026
**Status:** ✅ Ready to use - focuses on behavior, not styling
