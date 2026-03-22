# Updated Workflow (v2.1)

## ✅ Changes Applied

Based on your feedback, I've updated the workflow:

### **1. JIRA Integration** ✅
**Old:** Manual `/post-to-jira` command
**New:** Automatic prompt after `/verify-ac` completion

**Flow:**
```
/verify-ac EPS-1234
→ Verification completes
→ Shows report
→ Asks: "Post results to JIRA? (Y/n)"
→ If yes: Automatically posts to JIRA
→ If no: "You can post later with /post-to-jira"
```

### **2. Framework Selection** ✅
**Old:** Multi-select menu with all frameworks
**New:** Simple selection based on detected frameworks

**Flow:**
```
/generate-e2e-tests EPS-1234
→ Scans your project
→ Shows: "Detected: Playwright ✅, Jest ✅, Supertest ✅"
→ Asks: "Which framework? (1. Playwright, 2. Jest, 3. Supertest)"
→ User picks: "1,3" (Playwright + Supertest)
→ Generates only selected tests
```

### **3. Smart Selector Scan** ✅
**Old:** Separate `/smart-selector-scan` command
**New:** Integrated into `/generate-e2e-tests` (runs automatically)

**Flow:**
```
/generate-e2e-tests EPS-1234
→ STEP 1: Load ACs
→ STEP 2: 🔍 Smart Selector Scan (automatic)
   ✓ Scans src/ for selectors
   ✓ Builds selector map
   ✓ Shows quality: "87% coverage"
→ STEP 3: Detect tech stack
→ STEP 4: Choose framework (user selection)
→ STEP 5: Generate tests (using real selectors)
```

---

## 📊 Updated Complete Workflow

```bash
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 1: Start Task
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/collect-ac EPS-1234
→ Fetches from JIRA + Confluence + Figma
→ Smart AC detection
→ Creates AC checklist


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 2: Generate Tests
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/generate-e2e-tests EPS-1234

What happens automatically:
  1. 🔍 Scans for selectors (automatic)
     ✓ Found: 342 selectors
     ✓ Quality: 87%
  
  2. 📊 Detects tech stack
     ✓ Frontend: Next.js 14
     ✓ Installed: Playwright, Jest, Supertest
  
  3. ❓ Asks which framework:
     "Select: 1. Playwright ✅, 2. Jest ✅, 3. Supertest ✅"
     You pick: "1,3"
  
  4. ✅ Generates tests using REAL selectors:
     - tests/e2e/EPS-1234.spec.ts (Playwright)
     - tests/api/EPS-1234.test.ts (Supertest)
     Uses: [data-testid="submit-button"] ← from scan


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 3: Implement Feature
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# ... write your code ...


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 4: Run Tests
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

npx playwright test
npm test


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 5: Verify ACs
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/verify-ac EPS-1234

What happens:
  1. Interactive Q&A for each AC
  2. Generates verification report
  3. Shows: "Ready for PR? YES ✅"
  
  4. Automatically asks:
     "Post results to JIRA? (Y/n)"
     
     If YES:
       → Posts to JIRA ticket
       → Updates status
       → Adds labels
       → Shows: "✅ Posted to JIRA: [link]"
     
     If NO:
       → "Run /post-to-jira later if needed"


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# STEP 6: Create PR
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

gh pr create
→ Link verification report
```

---

## 🎯 Key Improvements

### **Better UX:**
✅ **Less manual steps** - Selector scan is automatic
✅ **Smart defaults** - Only shows installed frameworks
✅ **Streamlined flow** - JIRA post is part of verify-ac
✅ **User control** - Can say "no" to JIRA post

### **More Accurate:**
✅ **Real selectors** - Scan happens before test generation
✅ **Quality feedback** - Shows selector coverage %
✅ **Source comments** - Tests include selector locations

### **Simplified:**
```
Before: 5 separate commands
/collect-ac
/smart-selector-scan     ← Manual
/generate-e2e-tests
/verify-ac
/post-to-jira            ← Manual

After: 3 commands
/collect-ac
/generate-e2e-tests      ← Includes selector scan
/verify-ac               ← Includes JIRA post prompt
```

---

## 📋 Updated Command Sequence

### **Minimal Workflow:**
```bash
/collect-ac EPS-1234
# → Multi-source AC detection

/generate-e2e-tests EPS-1234
# → Selector scan (auto)
# → Framework selection (ask)
# → Test generation

# ... implement feature ...

/verify-ac EPS-1234
# → Verify ACs
# → Post to JIRA (ask)
```

### **Full Workflow with All Features:**
```bash
# 1. Start with ACs
/collect-ac EPS-1234

# 2. Optional: Add Figma specs
/figma-ac-extractor https://figma.com/...

# 3. Generate tests (includes selector scan)
/generate-e2e-tests EPS-1234
   → Selector scan runs automatically
   → Choose: Playwright + Supertest
   → Tests generated with real selectors

# 4. Code your feature
# ... implement ...

# 5. Run tests
npx playwright test

# 6. Verify (includes JIRA post)
/verify-ac EPS-1234
   → Verification complete
   → "Post to JIRA? (Y/n)" → Yes
   → ✅ Posted automatically

# 7. Create PR
gh pr create

# 8. End of sprint: Track quality
/ac-quality-trends
```

---

## 🔧 What Changed in Commands

### **`/verify-ac`** - Added JIRA Integration
```diff
+ Step 8: Offer to Post to JIRA
+ 
+ After verification, ask:
+ "Post results to JIRA? (Y/n)"
+ 
+ If yes → Automatically posts
+ If no → User can run /post-to-jira later
```

### **`/generate-e2e-tests`** - Integrated Selector Scan
```diff
+ Step 2: Smart Selector Scan (Automatic)
+ → Runs before framework selection
+ → Shows: "Found 342 selectors, 87% quality"
+ → Saves to: .ac-verification/selectors.json
+ 
  Step 3: Choose Test Framework (Simplified)
- Multi-select menu with 17 options
+ Simple selection from detected frameworks
+ "Detected: Playwright ✅, Jest ✅"
+ "Choose: 1, 2, or 1,2"
+ 
+ Step 7: Generate Tests Using Selector Map
+ → Uses real selectors from scan
+ → Adds source comments
+ → Shows quality in summary
```

### **`/smart-selector-scan`** - Still Available Separately
- Can still run standalone if needed
- But integrated into `/generate-e2e-tests` by default
- Useful for: checking quality, fixing issues

---

## 📊 Benefits

**Time Saved:**
- No manual selector scan step
- No manual JIRA update step
- Simpler framework selection

**Better Quality:**
- Tests use real selectors (not generic)
- Selector quality visible upfront
- Source tracking in test comments

**User Control:**
- Can decline JIRA post (stays local)
- Choose only needed frameworks
- See quality before generating

**Simplified Mental Model:**
```
Old: "Need to remember 5 commands"
New: "Just 3 commands, rest is automatic"
```

---

## 📍 Updated Files

**Modified:**
- `.claude/commands/verify-ac/verify-ac.md`
- `.claude/commands/generate-e2e-tests/generate-e2e-tests.md`

**Unchanged:**
- `/collect-ac` - Already has multi-source detection
- `/post-to-jira` - Still available standalone
- `/smart-selector-scan` - Still available if needed
- `/figma-ac-extractor` - Optional enhancement
- `/ac-quality-trends` - Sprint reporting

---

## ✅ Ready to Use

All changes applied! The workflow is now:

1. **Streamlined** - Fewer manual steps
2. **Automatic** - Selector scan runs auto
3. **Interactive** - Prompts at right time
4. **Flexible** - User can say "no" to JIRA

Try it:
```bash
cd /Users/mde/Documents/spectest
/collect-ac EPS-1234
/generate-e2e-tests EPS-1234  # ← Now includes selector scan
/verify-ac EPS-1234           # ← Now prompts for JIRA
```

**Updated: March 22, 2026**
