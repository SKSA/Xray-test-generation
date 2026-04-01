# AC Workflow - Index

## 📁 Location
**Base Directory:** `/Users/mde/Documents/spectest/`

---

## 📚 Documentation

### Main Documentation
- **[AC-WORKFLOW-README.md](./AC-WORKFLOW-README.md)** - Complete workflow guide (15KB)
  - Overview and problem statement
  - Complete developer workflow
  - Command details and usage
  - Best practices
  - Integration guides
  - Troubleshooting

### Quick Reference
- **[QUICK-START.md](./QUICK-START.md)** - 5-minute quick reference (4KB)
  - Command table
  - Basic workflow
  - Common issues

### Technical Documentation
- **[GENERATE-E2E-TESTS-ALIGNMENT.md](./GENERATE-E2E-TESTS-ALIGNMENT.md)** - spec-machine alignment analysis (12KB)
  - How `/generate-e2e-tests` aligns with spec-machine patterns
  - Testing conventions comparison
  - Recommendations for correctness
  - Project pattern detection

### Feature Documentation
- **[IMPROVEMENTS-V2.md](./IMPROVEMENTS-V2.md)** - Feature descriptions (17KB)
  - Detailed feature descriptions
  - Multi-source AC detection
  - Smart selector scanning
  - Quality tracking

### Integration Guides
- **[FIGMA-AC-FOCUS.md](./FIGMA-AC-FOCUS.md)** - Figma integration guide (5KB)
  - How Figma extraction works
  - User flow detection
  - Visual AC capture

- **[HOW-TO-SHARE.md](./HOW-TO-SHARE.md)** - Sharing guide (8KB)
  - How to share with your team
  - Installation instructions
  - Repository setup

---

## ⚙️ Command Files

Located in: `.claude/commands/`

### 1. Start Task Command
**File:** `.claude/commands/collect-ac/collect-ac.md`

**Purpose:** Initialize task with acceptance criteria

**Usage:** `/collect-ac TICKET-ID`

**Creates:**
- `.ac-verification/TICKET-ID/ac-checklist.md`

### 2. Verify AC Command
**File:** `.claude/commands/verify-ac/verify-ac.md`

**Purpose:** Verify AC testing status before PR

**Usage:** `/verify-ac TICKET-ID`

**Creates:**
- `.ac-verification/TICKET-ID/ac-checklist.md` (updated)
- `.ac-verification/TICKET-ID/verification-report.md`

### 3. Generate E2E Tests Command
**File:** `.claude/commands/generate-e2e-tests/generate-e2e-tests.md`

**Purpose:** Generate Playwright E2E tests from ACs

**Usage:** `/generate-e2e-tests TICKET-ID`

**Creates:**
- `tests/e2e/TICKET-ID.spec.ts`
- `tests/e2e/TICKET-ID-README.md`
- `playwright.config.ts` (if needed)

---

## 📊 Example Test Output

Located in: `tests/e2e/`

- **[google-homepage.spec.ts](./tests/google-homepage.spec.ts)** (192 lines)
  - 6 complete Playwright tests
  - Generated from test plan spec
  - Includes happy path + 5 edge cases

---

## 🚀 Quick Start

```bash
# 1. Copy commands to your repo
cp -r /Users/mde/Documents/spectest/.claude/commands \
      your-repo/.claude/

# 2. Use in spec-machine
/collect-ac EPS-1234
/generate-e2e-tests EPS-1234
/verify-ac EPS-1234

# 3. Create PR with verification report
gh pr create
```

---

## 📂 Full Directory Structure

```
/Users/mde/Documents/spectest/
│
├── 📖 Documentation
│   ├── AC-WORKFLOW-README.md      # Complete guide (READ THIS FIRST)
│   ├── QUICK-START.md             # Quick reference
│   └── INDEX.md                   # This file
│
├── ⚙️ Commands
│   └── .claude/commands/
│       ├── collect-ac/
│       │   └── collect-ac.md
│       ├── verify-ac/
│       │   └── verify-ac.md
│       └── generate-e2e-tests/
│           └── generate-e2e-tests.md
│
├── 🎭 Example Tests
│   ├── tests/e2e/
│   │   └── google-homepage.spec.ts
│   └── playwright.config.ts
│
└── 📋 Spec-Machine Integration
    ├── spec-machine/
    │   ├── config.yml
    │   ├── tech-stack.md
    │   └── specs/...
    └── package.json
```

---

## 🎯 Workflow Summary

```
┌──────────────────────────────────────────────────┐
│                                                  │
│  1. /collect-ac TICKET                          │
│     → Creates AC checklist                       │
│                                                  │
│  2. CODE YOUR FEATURE                            │
│     → Write implementation                       │
│                                                  │
│  3. /generate-e2e-tests TICKET                  │
│     → Auto-generates Playwright tests           │
│                                                  │
│  4. npx playwright test                         │
│     → Run E2E tests                             │
│                                                  │
│  5. /verify-ac TICKET                           │
│     → Verify AC status                          │
│     → Get "Ready for PR?" decision              │
│                                                  │
│  6. gh pr create                                │
│     → Link verification report                  │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 🔗 Related Resources

- **Main spec-machine repo:** [github.com/hellofresh/spec-machine](https://github.com/hellofresh/spec-machine)
- **Playwright docs:** [playwright.dev](https://playwright.dev)
- **Original conversation:** Agent transcript `5ebfcaf2-aea9-45db-a760-0b40ce00d245`

---

## 📞 Getting Help

**Questions about:**
- **Workflow:** See `AC-WORKFLOW-README.md`
- **Quick usage:** See `QUICK-START.md`
- **Commands:** See individual `.md` files in `.claude/commands/`
- **Examples:** See `tests/e2e/google-homepage.spec.ts`

---

## ✅ Next Steps

1. **Read the full guide:**
   ```bash
   cat AC-WORKFLOW-README.md
   ```

2. **Try it in your repo:**
   ```bash
   cp -r .claude/commands your-repo/.claude/
   cd your-repo
   /collect-ac YOUR-TICKET
   ```

3. **Integrate with CI/CD:**
   - See "Integration with CI/CD" section in main README

4. **Contribute back to spec-machine:**
   - Submit PR to main repo
   - Share with team

---

**Created:** March 22, 2026
**Location:** `/Users/mde/Documents/spectest/`
**Author:** Based on conversation about QA workflow automation
