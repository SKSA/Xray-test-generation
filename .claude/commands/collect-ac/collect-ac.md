---
name: /collect-ac
argument-hint: '[JIRA-TICKET]'
description: Collect acceptance criteria from JIRA, Confluence, and Figma
mode: single-agent
dependencies:
  - jira CLI (optional)
  - Confluence MCP (optional)
---

# Collect Acceptance Criteria

## Purpose
Initialize a task by intelligently fetching acceptance criteria from multiple sources: JIRA tickets, Confluence pages, Google Docs, or manual entry. Uses smart AC detection to parse various formats.

## Prerequisites Check

Check for integrations:
- JIRA CLI: `which jira`
- Confluence MCP: Check if Atlassian MCP is available
- Google Docs MCP: Check if Google Workspace MCP is available

## Workflow

### Step 1: Get Ticket ID

Ask user: "What's your JIRA ticket number? (e.g., EPS-1234)"

Store as `$TICKET`

### Step 2: Smart Source Detection

Try to fetch ACs from multiple sources in order:

#### A. JIRA Custom Field (Highest Priority)

If JIRA CLI available:
```bash
jira issue view $TICKET --plain
```

Look for custom field: "Acceptance Criteria"
```json
{
  "fields": {
    "customfield_10050": "AC content..."
  }
}
```

#### B. JIRA Description with Smart Parsing

Parse description for AC sections:

**Patterns to detect:**
1. **Explicit headers:**
   - "Acceptance Criteria"
   - "AC:"
   - "Acceptance:"
   - "Definition of Done"
   - "Success Criteria"

2. **BDD Format:**
   ```
   Given [context]
   When [action]
   Then [outcome]
   ```

3. **Numbered lists:**
   ```
   1. User can...
   2. System must...
   3. Page should...
   ```

4. **Bullet lists:**
   ```
   - User receives email
   - Code expires after 15 min
   * Success message shown
   ```

5. **Checkbox lists:**
   ```
   - [ ] Feature works on mobile
   - [ ] Accessible to screen readers
   ```

6. **Table format:**
   ```
   | # | Acceptance Criterion | Priority |
   |---|---------------------|----------|
   | 1 | User can login      | High     |
   ```

#### C. Linked Confluence Page

Check JIRA ticket for Confluence links:
- Field: "Confluence Page"
- Description links: `https://hellofresh.atlassian.net/wiki/spaces/...`
- Comments with Confluence links

If found, ask: "Found Confluence link. Fetch ACs from there? (Y/n)"

If yes:
```javascript
// Use Atlassian MCP to fetch page
const page = await atlassianMcp.getConfluencePage(pageId);
```

Parse Confluence content:
- **Headers:** `h1. Acceptance Criteria`, `h2. AC`, etc.
- **Lists:** Bullet points, numbered lists
- **Tables:** Extract from table columns
- **Panels:** Confluence info/note panels
- **Expand sections:** Content in collapsible sections

#### D. Linked Google Doc

Check for Google Docs links:
- `https://docs.google.com/document/d/...`

If found, ask: "Found Google Doc link. Fetch ACs from there? (Y/n)"

If yes:
```javascript
// Use Google Workspace MCP to fetch doc
const doc = await googleMcp.getDocument(docId);
```

Parse Google Doc:
- Headers with "AC" or "Acceptance"
- Lists (bulleted, numbered)
- Tables
- Comments/suggestions (optional)

#### E. JIRA Sub-tasks

Check if ticket has sub-tasks:
```bash
jira issue list --parent $TICKET
```

Ask: "Found 3 sub-tasks. Use as ACs? (Y/n)"

If yes, each sub-task summary becomes an AC:
```
Sub-task EPS-1234-1: "Implement email validation"
→ AC1: User email must be validated before submission
```

#### F. JIRA Comments

Check recent comments for AC clarifications:
```bash
jira issue comment list $TICKET
```

Look for comments containing:
- "AC clarification:"
- "Updated acceptance criteria:"
- "Additional AC:"

Parse and merge with existing ACs.

### Step 3: Smart AC Detection & Parsing

For each detected AC source, apply smart parsing:

**Algorithm:**
```python
def parse_acceptance_criteria(text):
    acs = []
    
    # 1. Find AC sections
    sections = find_sections_with_keywords(text, [
        "acceptance criteria", "ac:", "given when then",
        "definition of done", "success criteria"
    ])
    
    # 2. Extract structured content
    for section in sections:
        # Try BDD format first
        bdd_acs = extract_bdd_format(section)
        if bdd_acs:
            acs.extend(bdd_acs)
            continue
        
        # Try lists (numbered, bullets, checkboxes)
        list_acs = extract_lists(section)
        if list_acs:
            acs.extend(list_acs)
            continue
        
        # Try tables
        table_acs = extract_tables(section)
        if table_acs:
            acs.extend(table_acs)
            continue
        
        # Fallback: split by sentences/paragraphs
        acs.extend(extract_sentences(section))
    
    # 3. Clean and normalize
    acs = [clean_ac(ac) for ac in acs]
    
    # 4. Deduplicate
    acs = remove_duplicates(acs)
    
    # 5. Number sequentially
    acs = [(i+1, ac) for i, ac in enumerate(acs)]
    
    return acs
```

**Cleaning rules:**
- Remove markdown formatting (* _ ` etc.)
- Remove JIRA markup ({color}, {noformat}, etc.)
- Trim whitespace
- Remove empty lines
- Normalize "[ ]" to "- [ ]"
- Remove ticket references (e.g., "See EPS-999")

### Step 4: Display Detected ACs

Show all detected ACs with source:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Detected Acceptance Criteria
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Source: JIRA Description + Confluence Page

AC #1: User receives verification email within 30 seconds
  Source: JIRA description (line 15)
  Format: User story

AC #2: Given user enters invalid email
       When user submits form
       Then error message is displayed
  Source: Confluence page "Email Feature Spec"
  Format: BDD (Given/When/Then)

AC #3: Code expires after 15 minutes
  Source: JIRA comment by alice@hf.com (2026-03-20)
  Format: Sentence

AC #4: Success notification shown
  Source: Sub-task EPS-1234-2
  Format: Task summary

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total: 4 acceptance criteria detected
```

### Step 5: Review & Edit

Ask user: "Are these ACs correct? (Y/n/edit)"

**Options:**

1. **Yes** - Accept all ACs as-is

2. **No** - Go to manual entry

3. **Edit** - Interactive editing:
   ```
   Which AC to edit? (1-4, or 'add' for new, 'delete' to remove)
   > 2
   
   Current AC #2:
   "Given user enters invalid email
    When user submits form
    Then error message is displayed"
   
   Edit (or press Enter to keep):
   > [User types new text]
   
   ✅ Updated AC #2
   ```

### Step 6: Manual Entry (Fallback)

If no ACs detected or user chooses manual entry:

Ask user to enter each AC with smart formatting assistance:

```
Enter AC #1 (or 'done' to finish):

Choose format:
1. Given/When/Then (BDD)
2. User can... (User story)
3. System must... (System requirement)
4. As a [role], I want [goal], so that [benefit]
5. Custom (type your own)

Your choice: 1

Given: [User types context]
When: [User types action]
Then: [User types outcome]

Preview:
"Given user is logged in
 When user clicks update button
 Then profile is saved and success message shown"

Correct? (Y/n)
```

### Step 7: Create AC Checklist

Create directory: `.ac-verification/$TICKET/`

Create file: `.ac-verification/$TICKET/ac-checklist.md`

**Enhanced format with metadata:**
```markdown
# Acceptance Criteria Checklist

**Ticket:** $TICKET
**Created:** 2026-03-22 10:30:00
**Created By:** mde@hellofresh.com
**Sources:** JIRA description, Confluence page "Email Feature Spec"

## Acceptance Criteria

### AC #1: User receives verification email within 30 seconds
**Source:** JIRA description (line 15)
**Format:** User story
**Priority:** High
**Test Type:** E2E, API
- [ ] Tested and PASSED
- [ ] Evidence: (attach screenshot/link)

### AC #2: Code expires after 15 minutes
**Source:** Confluence page
**Format:** BDD
**Priority:** High
**Test Type:** E2E, Unit
- [ ] Tested and PASSED
- [ ] Evidence: (attach screenshot/link)

### AC #3: Success notification shown
**Source:** JIRA comment
**Format:** Sentence
**Priority:** Medium
**Test Type:** E2E
- [ ] Tested and PASSED
- [ ] Evidence: (attach screenshot/link)

---

## Metadata

**Detected Sources:**
- JIRA ticket EPS-1234
- Confluence page: https://...
- JIRA comments (1 comment)
- Sub-tasks (1 sub-task)

**Detection Confidence:** High (4/4 ACs from structured sources)

**Suggested Test Frameworks:**
- Playwright (E2E)
- Jest (Unit tests for expiration logic)
- Supertest (API tests for email sending)

---
Created: 2026-03-22 10:30:00
Last Updated: 2026-03-22 10:30:00
```

### Step 8: Confidence Score

Display detection confidence:

```
📊 AC Detection Quality Score

Confidence: ⭐⭐⭐⭐⭐ (High)

Breakdown:
✅ All ACs from structured sources (JIRA, Confluence)
✅ Consistent formatting detected
✅ No ambiguous language found
✅ Sources verified and linked

⚠️ Suggestions:
- AC #3 could be more specific (what notification?)
- Consider edge case: what if email delivery fails?
```

### Step 9: Link Back to Sources

Store source links in: `.ac-verification/$TICKET/sources.json`

```json
{
  "ticket": "EPS-1234",
  "sources": [
    {
      "type": "jira-description",
      "url": "https://hellofresh.atlassian.net/browse/EPS-1234",
      "acs": [1, 2]
    },
    {
      "type": "confluence",
      "url": "https://hellofresh.atlassian.net/wiki/spaces/ENG/pages/123456",
      "title": "Email Feature Spec",
      "acs": [3, 4]
    },
    {
      "type": "jira-comment",
      "author": "alice@hf.com",
      "date": "2026-03-20",
      "acs": [5]
    }
  ]
}
```

### Step 10: Confirmation

Display summary:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Task Started: EPS-1234
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Sources Detected:
✅ JIRA description
✅ Confluence page: "Email Feature Spec"
✅ JIRA comments (1)
✅ Sub-tasks (1)

Acceptance Criteria: 4
  AC1: User receives verification email
  AC2: Code expires after 15 minutes
  AC3: Success notification shown
  AC4: Accessible on mobile devices

Detection Confidence: ⭐⭐⭐⭐⭐ High

📝 Next Steps:
  1. Review: .ac-verification/EPS-1234/ac-checklist.md
  2. Create branch: git checkout -b feature/EPS-1234
  3. Generate tests: /generate-e2e-tests EPS-1234
  4. Implement feature
  5. Verify: /verify-ac EPS-1234

✅ AC checklist saved to: .ac-verification/EPS-1234/
```

## Advanced Features

### A. Figma Integration

If Figma links found in JIRA/Confluence:
```
Found Figma designs:
https://figma.com/file/xyz/Email-Flow

Extract visual ACs? (Y/n)
→ "Button must be 44px high (touch target)"
→ "Loading spinner shown during send"
→ "Success checkmark animated"
```

### B. Historical AC Learning

Learn from past tickets:
```
💡 Based on similar tickets (EPS-999, EPS-1001):
Common missing ACs:
- Error handling for network issues
- Loading states
- Empty states

Add these? (y/N)
```

### C. AC Templates

Suggest templates based on ticket type:
```
Ticket labels: "authentication", "form"

Suggested AC template: Authentication Form
- User can see password strength indicator
- User can toggle password visibility
- Form validates on blur
- Error messages shown inline

Use this template? (y/N)
```

## Output Files

- `.ac-verification/$TICKET/ac-checklist.md` - AC checklist
- `.ac-verification/$TICKET/sources.json` - Source links
- `.ac-verification/$TICKET/detection-log.txt` - Parsing debug log
- `.ac-verification/$TICKET/raw-content.txt` - Raw source content

## Error Handling

| Issue | Solution |
|-------|----------|
| No ACs detected | Prompt for manual entry |
| Multiple conflicting ACs | Show all, let user choose |
| Malformed BDD | Suggest corrections |
| Broken Confluence link | Fall back to JIRA only |
| No JIRA access | Use manual entry mode |
