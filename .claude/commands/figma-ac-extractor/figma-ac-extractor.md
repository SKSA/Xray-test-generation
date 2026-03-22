---
name: /figma-ac-extractor
argument-hint: '[figma-url]'
description: Extract acceptance criteria and visual requirements from Figma designs
mode: single-agent
dependencies:
  - Figma MCP or Figma API access
---

# Figma AC Extractor

## Purpose
Automatically extract **user flow and interaction acceptance criteria** from Figma designs to supplement JIRA ticket ACs. Focuses on user actions, navigation, and critical paths - NOT visual styling like colors or font sizes.

## Prerequisites

Check for Figma integration:
```bash
# Check for Figma MCP
ls ~/.cursor/mcp/servers/ | grep figma

# Or check for Figma plugin
# Or use Figma API token
```

## Workflow

### Step 1: Get Figma URL

Ask user: "Paste your Figma design URL:"

Example: `https://www.figma.com/design/xyz123/Email-Flow?node-id=1-123`

Parse URL to extract:
- File ID: `xyz123`
- Node ID: `1-123` (specific frame/component)
- Frame name: `Email Flow`

### Step 2: Fetch Figma Data

Using Figma API or MCP:

```typescript
const figmaData = await figma.getFile(fileId);
const node = figma.getNode(fileId, nodeId);

// Extract:
const components = node.children;
const styles = node.styles;
const variables = node.componentVariables;
const annotations = node.devResources; // Dev Mode comments
```

### Step 3: Extract User Flow & Interaction ACs

**IMPORTANT: Focus on behavior, NOT styling**

**Skip extracting:**
- ❌ Colors (hex codes, RGB values)
- ❌ Font sizes, font families
- ❌ Exact dimensions (px values)
- ❌ Spacing, padding, margins
- ❌ Border radius, shadows

**DO extract:**
- ✅ User interactions (clicks, taps, swipes)
- ✅ Navigation flows (screen transitions)
- ✅ State changes (enabled/disabled, loading)
- ✅ Critical user paths
- ✅ Form behaviors (validation, submission)
- ✅ Conditional logic (show/hide, permissions)

**A. Component Detection (For Interactions Only)**
```typescript
// Find interactive components only
const interactiveComponents = [];

for (const child of node.children) {
  // Only track components that have interactions
  if (child.interactions && child.interactions.length > 0) {
    interactiveComponents.push({
      name: child.name,
      type: child.type,
      interactions: child.interactions,
      states: child.componentProperties?.state || []
    });
  }
}
```

**B. Extract Interaction ACs (Primary Focus)**
```typescript
// From Figma interactions/prototyping
function extractInteractionAC(component) {
  const interactions = component.interactions || [];
  const acs = [];
  
  interactions.forEach(interaction => {
    // Map Figma triggers to user actions
    const triggerMap = {
      'ON_CLICK': 'clicks',
      'ON_HOVER': 'hovers over',
      'ON_PRESS': 'presses',
      'MOUSE_ENTER': 'hovers over',
      'MOUSE_LEAVE': 'stops hovering',
      'ON_DRAG': 'drags'
    };
    
    const actionMap = {
      'NAVIGATE': `navigate to ${interaction.destination?.name}`,
      'CHANGE_TO': `change to ${interaction.destinationState} state`,
      'OPEN_OVERLAY': `open ${interaction.destination?.name} overlay`,
      'CLOSE_OVERLAY': 'close overlay',
      'BACK': 'navigate back',
      'SCROLL_TO': `scroll to ${interaction.destination?.name}`
    };
    
    const trigger = triggerMap[interaction.trigger] || interaction.trigger.toLowerCase();
    const action = actionMap[interaction.action] || interaction.action.toLowerCase();
    
    acs.push({
      type: 'interaction',
      ac: `When user ${trigger} ${component.name}, ${action}`,
      category: 'user-flow',
      testable: true,
      e2eTest: true,
      criticalPath: isInMainFlow(component)
    });
  });
  
  // Extract state-based behaviors
  if (component.componentProperties?.state) {
    const states = component.componentProperties.state;
    states.forEach(state => {
      acs.push({
        type: 'state',
        ac: `${component.name} must have ${state} state`,
        category: 'user-flow',
        testable: true
      });
    });
  }
  
  return acs;
}

// Identify critical paths
function isInMainFlow(component) {
  const criticalKeywords = ['submit', 'continue', 'next', 'checkout', 'confirm', 'save', 'login', 'signup'];
  return criticalKeywords.some(keyword => 
    component.name.toLowerCase().includes(keyword)
  );
}
```

**C. Extract Form Behavior ACs**
```typescript
function extractFormBehaviorAC(component) {
  const acs = [];
  
  // Detect form fields
  if (component.type === 'INSTANCE' && component.name.includes('Input')) {
    // Required field detection
    if (component.name.includes('Required') || component.name.includes('*')) {
      acs.push({
        type: 'validation',
        ac: `${component.name} is required and must show error if empty`,
        category: 'form-behavior',
        testable: true
      });
    }
    
    // Email validation
    if (component.name.toLowerCase().includes('email')) {
      acs.push({
        type: 'validation',
        ac: `${component.name} must validate email format`,
        category: 'form-behavior',
        testable: true
      });
    }
    
    // Password requirements
    if (component.name.toLowerCase().includes('password')) {
      acs.push({
        type: 'validation',
        ac: `${component.name} must meet password requirements`,
        category: 'form-behavior',
        testable: true
      });
    }
  }
  
  // Submit button detection
  if (component.name.toLowerCase().includes('submit')) {
    acs.push({
      type: 'interaction',
      ac: `${component.name} must be disabled until form is valid`,
      category: 'form-behavior',
      testable: true
    });
    
    acs.push({
      type: 'interaction',
      ac: `${component.name} must show loading state during submission`,
      category: 'form-behavior',
      testable: true
    });
  }
  
  return acs;
}
```

**D. Extract Functional Accessibility ACs (Behavior Only)**
```typescript
function extractFunctionalA11yAC(component) {
  const a11yACs = [];
  
  // Skip visual-only a11y (color contrast, sizes)
  // Focus on functional a11y only
  
  // Keyboard navigation
  if (component.type === 'BUTTON' || component.type === 'INSTANCE') {
    a11yACs.push({
      type: 'accessibility',
      ac: `${component.name} must be keyboard accessible (Tab, Enter/Space)`,
      category: 'a11y-functional',
      testable: true,
      wcag: '2.1.1'
    });
  }
  
  // Focus management
  if (component.interactions?.some(i => i.action === 'OPEN_OVERLAY')) {
    a11yACs.push({
      type: 'accessibility',
      ac: `When ${component.name} opens overlay, focus must move to overlay`,
      category: 'a11y-functional',
      testable: true,
      wcag: '2.4.3'
    });
  }
  
  // Screen reader labels (if missing text)
  if (!component.children?.some(c => c.type === 'TEXT')) {
    a11yACs.push({
      type: 'accessibility',
      ac: `${component.name} must have accessible label for screen readers`,
      category: 'a11y-functional',
      testable: true,
      wcag: '4.1.2'
    });
  }
  
  // Form error announcements
  if (component.name.toLowerCase().includes('error')) {
    a11yACs.push({
      type: 'accessibility',
      ac: `${component.name} must be announced to screen readers when shown`,
      category: 'a11y-functional',
      testable: true,
      wcag: '3.3.1'
    });
  }
  
  return a11yACs;
}
```

**E. Extract Critical User Paths**
```typescript
function extractCriticalPaths(figmaData) {
  const paths = [];
  const startingScreens = findScreens(figmaData, ['home', 'landing', 'login']);
  
  startingScreens.forEach(screen => {
    const path = tracePath(screen, []);
    if (path.length >= 2) { // Multi-step flow
      paths.push({
        type: 'critical-path',
        ac: `User flow: ${path.map(s => s.name).join(' → ')}`,
        category: 'user-journey',
        testable: true,
        e2eTest: true,
        steps: path.length
      });
    }
  });
  
  return paths;
}

function tracePath(screen, visited) {
  if (visited.includes(screen.id)) return visited;
  
  const path = [...visited, screen];
  const interactions = screen.children.flatMap(c => c.interactions || []);
  
  // Follow the main action (usually primary button)
  const mainAction = interactions.find(i => 
    i.trigger === 'ON_CLICK' && 
    i.destination &&
    !visited.includes(i.destination.id)
  );
  
  if (mainAction?.destination) {
    return tracePath(mainAction.destination, path);
  }
  
  return path;
}
```

### Step 4: Parse Dev Mode Comments (Behavioral Notes Only)

Figma Dev Mode allows designers to add implementation notes:

```typescript
// Extract behavioral annotations from Dev Mode
const devNotes = figmaData.devResources || [];

devNotes.forEach(note => {
  if (note.type === 'comment') {
    // Parse comment for behavioral ACs
    const acMatch = note.text.match(/AC:\s*(.+)/i);
    if (acMatch) {
      const acText = acMatch[1];
      
      // Filter out visual-only ACs
      const visualKeywords = ['color', 'font', 'size', 'px', 'spacing', 'padding', 'margin', 'border'];
      const isBehavioral = !visualKeywords.some(keyword => 
        acText.toLowerCase().includes(keyword)
      );
      
      if (isBehavioral) {
        acs.push({
          type: 'design-note',
          ac: acText,
          category: 'designer-specified',
          author: note.author.name,
          priority: 'high'
        });
      }
    }
    
    // Parse for behavioral implementation notes
    const implNote = note.text.match(/Implementation:\s*(.+)/i);
    if (implNote && !implNote[1].toLowerCase().includes('style')) {
      implementationNotes.push(implNote[1]);
    }
    
    // Parse for user flow notes
    const flowNote = note.text.match(/Flow:\s*(.+)/i);
    if (flowNote) {
      acs.push({
        type: 'user-flow',
        ac: flowNote[1],
        category: 'designer-specified',
        author: note.author.name,
        priority: 'high'
      });
    }
  }
});
```

### Step 5: Generate Behavioral AC List

Combine all extracted behavioral ACs:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Extracted from Figma: Email Verification Flow
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Source: https://figma.com/design/xyz123
Frame: Email Verification Flow
Last Modified: 2026-03-20 by alice@hellofresh.com

Focus: User flows, interactions, and critical paths
Excluded: Visual styling (colors, fonts, dimensions)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🛤️ Critical User Paths (2)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-P1: User flow: Landing → Email Input → Verification → Success
AC-P2: User flow: Landing → Email Input → Error → Retry

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🖱️ Interaction Acceptance Criteria (6)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-I1: When user clicks Submit button, navigate to Verification screen
AC-I2: When user hovers Submit button, show hover state
AC-I3: When form is submitting, button shows loading state
AC-I4: When error occurs, show error message
AC-I5: When user clicks Resend link, resend verification email
AC-I6: When user clicks Back button, navigate to previous screen

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Form Behavior Acceptance Criteria (5)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-F1: Email input is required and must show error if empty
AC-F2: Email input must validate email format
AC-F3: Submit button must be disabled until email is valid
AC-F4: Submit button must be disabled during submission
AC-F5: Error message must clear when user starts typing

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
♿ Accessibility Acceptance Criteria (5)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-A1: Submit button must be at least 44px high (touch target)
  Priority: HIGH | WCAG: 2.5.5
  
AC-A2: Text contrast ratio must be ≥ 4.5:1
  Priority: CRITICAL | WCAG: 1.4.3
  Current ratio: 8.2:1 ✅
  
AC-A3: Form must be keyboard navigable (tab order)
  Priority: HIGH | WCAG: 2.1.1
  
AC-A4: Error messages must be announced to screen readers
  Priority: HIGH | WCAG: 4.1.3
  
AC-A5: Success icon must have alt text "Success"
  Priority: MEDIUM | WCAG: 1.1.1

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💬 Designer Notes (2)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC-D1: Use slide-up animation for success notification (300ms ease-out)
  - From: Alice (2026-03-20)
  
AC-D2: Loading spinner uses primary color, 24px diameter
  - From: Bob (2026-03-21)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total ACs: 23
- Visual: 12 (dimensions, colors, typography, spacing)
- Interaction: 4 (user flows, states)
- Accessibility: 5 (WCAG compliance)
- Designer Notes: 2

Testable: 21/23 (91%)
- Visual Regression: 12 tests
- E2E Tests: 4 tests
- Accessibility Tests: 5 tests
```

### Step 7: Merge with Existing ACs

If `/collect-ac` already ran:

```
Found existing ACs from JIRA ticket EPS-1234.
Merge with Figma ACs? (Y/n)

Merging strategies:
1. Append (add Figma ACs as additional)
2. Replace visual ACs (keep functional, replace visual)
3. Manual merge (choose which to keep)

Your choice: 2

✅ Merged:
- Kept 4 functional ACs from JIRA
- Added 12 visual ACs from Figma
- Added 5 accessibility ACs from Figma
- Total: 21 ACs
```

### Step 8: Generate Visual Test Code

Auto-generate visual regression tests:

```typescript
import { test, expect } from '@playwright/test';

test.describe('EPS-1234: Visual Requirements', () => {
  
  /**
   * AC-V1: Email input must be 320px × 48px
   */
  test('AC-V1: Email input dimensions', async ({ page }) => {
    await page.goto('/signup');
    
    const input = page.locator('[data-testid="email-input"]');
    const box = await input.boundingBox();
    
    expect(box.width).toBeCloseTo(320, 1);
    expect(box.height).toBeCloseTo(48, 1);
  });
  
  /**
   * AC-V4: Submit button background must be #00A862
   */
  test('AC-V4: Button background color', async ({ page }) => {
    await page.goto('/signup');
    
    const button = page.locator('[data-testid="submit-button"]');
    const bgColor = await button.evaluate(el => 
      window.getComputedStyle(el).backgroundColor
    );
    
    // Convert rgb(0, 168, 98) to hex
    expect(rgbToHex(bgColor)).toBe('#00A862');
  });
  
  /**
   * AC-V7: Heading uses Inter Bold 24px
   */
  test('AC-V7: Heading typography', async ({ page }) => {
    await page.goto('/signup');
    
    const heading = page.locator('h1');
    const styles = await heading.evaluate(el => ({
      fontFamily: window.getComputedStyle(el).fontFamily,
      fontSize: window.getComputedStyle(el).fontSize,
      fontWeight: window.getComputedStyle(el).fontWeight
    }));
    
    expect(styles.fontFamily).toContain('Inter');
    expect(styles.fontSize).toBe('24px');
    expect(styles.fontWeight).toBe('700'); // Bold
  });
  
  /**
   * Complete visual regression
   */
  test('AC-Visual: Screenshot comparison', async ({ page }) => {
    await page.goto('/signup');
    
    // Compare against Figma design
    await expect(page).toHaveScreenshot('signup-page.png', {
      maxDiffPixels: 100, // Allow minor rendering differences
      threshold: 0.2
    });
  });
});
```

### Step 9: Create Figma Link in Checklist

Update AC checklist with Figma references:

```markdown
## Visual Design Reference

**Figma:** [Email Flow](https://figma.com/design/xyz123?node-id=1-123)
**Last Updated:** 2026-03-20 by alice@hellofresh.com
**Design Tokens:** `.ac-verification/EPS-1234/design-tokens.json`

### AC-V1: Email input dimensions (320px × 48px)
**Source:** Figma Frame "Email Input"
**Figma Node:** 1-456
**Screenshot:** ![Email Input](./figma-screenshots/email-input.png)
- [ ] Tested and PASSED
- [ ] Visual regression test passing

### AC-V4: Submit button color (#00A862)
**Source:** Figma Component "Primary Button"
**Design Token:** colors.primary
**Screenshot:** ![Button](./figma-screenshots/button.png)
- [ ] Tested and PASSED
```

### Step 10: Export Design Assets

Download reference images from Figma:

```typescript
// Export each component as PNG
for (const component of components) {
  const imageUrl = await figma.exportImage(component.id, {
    format: 'PNG',
    scale: 2 // Retina
  });
  
  // Save to .ac-verification/$TICKET/figma-screenshots/
  await downloadImage(imageUrl, `${component.name}.png`);
}
```

## Output Files

- `.ac-verification/$TICKET/figma-acs.md` - Extracted visual ACs
- `.ac-verification/$TICKET/design-tokens.json` - Design tokens
- `.ac-verification/$TICKET/figma-screenshots/` - Reference images
- `.ac-verification/$TICKET/component-specs.json` - Component specifications
- `tests/visual/$TICKET-visual.spec.ts` - Generated visual tests

## Integration

This integrates with:
- `/collect-ac` - Merge Figma ACs with JIRA ACs
- `/generate-e2e-tests` - Add visual regression tests
- `/verify-ac` - Include visual verification
- Design system tools - Validate design token usage

## Benefits

✅ Complete visual requirements coverage
✅ Designer-developer alignment
✅ Automated visual regression testing
✅ Design token validation
✅ Accessibility requirements from design
✅ Single source of truth (Figma → Tests)
