---
name: /verify-ac
argument-hint: '[JIRA-TICKET]'
description: Verify acceptance criteria testing status before PR
mode: single-agent
---

# Verify Acceptance Criteria

## Purpose
Confirm testing status for each acceptance criterion before creating a pull request. Ensures all ACs are either tested, marked as N/A, or explicitly deferred.

## Workflow

### Step 1: Load AC Data

Ask user: "What's your JIRA ticket number? (e.g., EPS-1234)"

Store as `$TICKET`

Read AC checklist from: `.ac-verification/$TICKET/ac-checklist.md`

If file doesn't exist:
- Error: "No AC checklist found for $TICKET. Run /collect-ac first."
- Exit

### Step 2: Display Current Status

Show all ACs with current status (checked or unchecked).

### Step 3: Verify Each AC

For each acceptance criterion:

Display: "AC #X: [criterion text]"

Ask user: "What's the testing status for this AC?"

Options:
1. **✅ Tested and PASSED** - I tested this and it works
2. **❌ Tested and FAILED** - I tested this and found issues
3. **🔍 Needs QA testing** - Requires QA team to verify
4. **N/A Not applicable** - This AC doesn't apply to my changes
5. **⏳ Will test later** - Deferred for now

Store response.

If "Tested and FAILED":
- Ask: "What issues did you find?"
- Record issues

### Step 4: Update AC Checklist

Update `.ac-verification/$TICKET/ac-checklist.md`:

```markdown
# Acceptance Criteria Checklist

Ticket: $TICKET
Created: YYYY-MM-DD

## Acceptance Criteria

- [x] AC1: [description] — Tested and PASSED
- [ ] AC2: [description] — Needs QA testing
- [x] AC3: [description] — Not applicable

---
Created: YYYY-MM-DD
Verified: YYYY-MM-DD
```

### Step 5: Generate Verification Report

Create: `.ac-verification/$TICKET/verification-report.md`

Format:
```markdown
# Verification Report: $TICKET

Generated: YYYY-MM-DD HH:MM

## Status Summary

| # | Acceptance Criterion | Status |
|---|---------------------|--------|
| 1 | [AC text] | ✅ Tested and PASSED |
| 2 | [AC text] | 🔍 Needs QA testing |
| 3 | [AC text] | N/A Not applicable |

## Statistics

- **Total AC:** X
- **Tested & Passed:** X
- **Tested & Failed:** X
- **Needs QA:** X
- **Not Applicable:** X
- **Deferred:** X

## Ready for PR?

[YES/NO based on criteria below]

### Criteria for "Ready for PR"
- All ACs are either:
  - ✅ Tested and PASSED, OR
  - 🔍 Needs QA testing, OR
  - N/A Not applicable
- No ACs are:
  - ❌ Tested and FAILED, OR
  - ⏳ Will test later (unless explicitly approved)

## Issues Found

[List any issues from failed tests]

## Next Steps

[Recommendations based on status]
```

### Step 6: Display Results

Show formatted table:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
$TICKET — Verification Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬──────────────────────────────────┬────────────────┐
│  #  │     Acceptance Criterion         │     Status     │
├─────┼──────────────────────────────────┼────────────────┤
│ 1   │ User can update profile          │ ✅ PASSED      │
│ 2   │ Success notification shows       │ 🔍 Needs QA    │
│ 3   │ Email validation works           │ ✅ PASSED      │
└─────┴──────────────────────────────────┴────────────────┘

Summary:
- Total AC: 3
- Tested & Passed: 2
- Needs QA: 1
- Not Applicable: 0
- Deferred: 0

Ready for PR? YES ✅

All critical acceptance criteria verified. You're clear to open a pull request.
```

### Step 7: Next Action Suggestion

Based on results:

**If Ready for PR:**
- "All ACs verified! Next steps:"
- "1. Run E2E tests: npx playwright test"
- "2. Create PR: gh pr create"
- "3. Link this verification report in PR description"

**If NOT Ready:**
- "⚠️ Cannot create PR yet. Issues found:"
- List blockers
- "Next steps:"
- List what needs to be fixed

### Step 8: Offer to Post to JIRA

After verification is complete, ask user:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Verification Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Post results to JIRA ticket $TICKET? (Y/n)

This will:
  • Post verification report as comment
  • Update ticket status (if all passed)
  • Add labels: ac-verified, e2e-tested
  • Mention assignee
```

If user says **Yes**:
```bash
# Automatically run /post-to-jira
→ Posts formatted comment to JIRA
→ Updates ticket status if ready
→ Adds labels
→ Confirms: "✅ Posted to JIRA: [link]"
```

If user says **No**:
```
"You can post later by running: /post-to-jira $TICKET"
```

## Output Files

- `.ac-verification/$TICKET/ac-checklist.md` - Updated checklist
- `.ac-verification/$TICKET/verification-report.md` - Verification report
- `.ac-verification/$TICKET/jira-comment.txt` - JIRA comment (if posted)
