# AC Workflow - Quick Reference

> **TL;DR:** Three commands to ensure ACs are tested before PR

---

## Commands

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/collect-ac TICKET` | Create AC checklist | Start of every task |
| `/generate-e2e-tests TICKET` | Generate Playwright tests | After coding (or before) |
| `/verify-ac TICKET` | Verify AC testing status | Before creating PR |

---

## 5-Minute Workflow

```bash
# 1. Start task
/collect-ac EPS-1234
# → Creates .ac-verification/EPS-1234/ac-checklist.md

# 2. Code your feature
git checkout -b feature/EPS-1234
# ... write code ...

# 3. Generate E2E tests
/generate-e2e-tests EPS-1234
# → Creates tests/e2e/EPS-1234.spec.ts

# 4. Run tests
npx playwright test

# 5. Verify ACs
/verify-ac EPS-1234
# → Interactive Q&A for each AC
# → Creates verification-report.md

# 6. Create PR
gh pr create
# → Link verification report in description
```

---

## Files Created

```
.ac-verification/
  EPS-1234/
    ├── ac-checklist.md          # AC tracking
    └── verification-report.md   # Final report (link in PR!)

tests/e2e/
  ├── EPS-1234.spec.ts          # E2E tests
  └── EPS-1234-README.md         # Test docs
```

---

## Testing Status Options

When running `/verify-ac`, you'll be asked for each AC:

- ✅ **Tested and PASSED** - It works
- ❌ **Tested and FAILED** - Found issues
- 🔍 **Needs QA testing** - QA will verify
- **N/A Not applicable** - Doesn't apply
- ⏳ **Will test later** - Deferred

---

## Ready for PR?

**YES if:**
- All ACs are: ✅ Passed, 🔍 Needs QA, or N/A
- No ACs are: ❌ Failed or ⏳ Deferred

**NO if:**
- Any AC is ❌ Failed or ⏳ Deferred

---

## Example Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EPS-1234 — Verification Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬───────────────────────────┬────────────┐
│  #  │   Acceptance Criterion    │   Status   │
├─────┼───────────────────────────┼────────────┤
│ 1   │ User receives email       │ ✅ PASSED  │
│ 2   │ Code expires after 15min  │ ✅ PASSED  │
│ 3   │ Max 3 resend attempts     │ 🔍 Needs QA│
└─────┴───────────────────────────┴────────────┘

Summary:
- Total AC: 3
- Tested & Passed: 2
- Needs QA: 1

Ready for PR? YES ✅
```

---

## Installation

Copy commands to your repo:

```bash
cp -r /Users/mde/Documents/spectest/.claude/commands/* \
      your-repo/.claude/commands/
```

Or add to `spec-machine.config.yml`:
```yaml
commands:
  - name: /collect-ac
  - name: /verify-ac
  - name: /generate-e2e-tests
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "No AC checklist found" | Run `/collect-ac` first |
| "JIRA CLI not found" | Enter ACs manually |
| Tests failing | Review selectors, run `npx playwright test --debug` |
| Can't create PR | Fix failing ACs, run `/verify-ac` again |

---

## Full Documentation

See: `AC-WORKFLOW-README.md` for complete guide

---

**Commands stored at:**
```
/Users/mde/Documents/spectest/.claude/commands/
├── collect-ac/collect-ac.md
├── verify-ac/verify-ac.md
└── generate-e2e-tests/generate-e2e-tests.md
```

**Documentation at:**
```
/Users/mde/Documents/spectest/AC-WORKFLOW-README.md
```
