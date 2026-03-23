# Changelog

All notable changes to the Spec Dev QA Assistant project.

## [v2.1.0] - March 22, 2026

### Added
- **Automatic JIRA Integration**: `/verify-ac` now prompts to post results to JIRA automatically
- **Platform Selection**: `/generate-e2e-tests` supports Web (Playwright/Cypress) and Mobile React Native (Maestro/Detox)
- **Automatic Dependency Management**: New `/setup-qa-assistant` command to detect and install missing dependencies for teammates
- **Integrated Selector Scan**: Smart selector scanning now runs automatically within `/generate-e2e-tests`

### Changed
- **Command Rename**: `/start-task` → `/collect-ac` for better clarity on what the command does
- **Workflow Order**: Corrected to "implement code first, then generate tests" (not vice versa)
- **Framework Selection**: Simplified to show only detected/installed frameworks instead of all 17 options
- **Figma AC Extractor**: Now focuses only on user flows, interactions, and critical paths (excludes visual styling like colors/fonts)
- **Repository Name**: Renamed from `dev-qa-assistant` → `spec-dev-qa-assistant` to better represent full capabilities

### Improved
- **Documentation Organization**: Moved all supporting docs to `/docs` folder
- **JIRA Workflow**: User now has control to accept/decline automatic JIRA posting
- **Test Generation**: Tests now use real selectors from codebase via automatic scanning
- **Quality Feedback**: Selector coverage % visible before test generation

### Fixed
- Workflow documentation now correctly shows "implement → test" order (not "test → implement")
- Command definitions updated to match corrected workflow
- Removed outdated `start-task` references

---

## Detailed v2.1 Changes

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
/start-task
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

# ... implement feature ...

/generate-e2e-tests EPS-1234
# → Selector scan (auto)
# → Framework selection (ask)
# → Test generation

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

# 3. Implement feature
# ... write your code ...

# 4. Generate tests (includes selector scan)
/generate-e2e-tests EPS-1234
   → Selector scan runs automatically
   → Choose: Playwright + Supertest
   → Tests generated with real selectors

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

## 📍 Modified Files

**Commands:**
- `.claude/commands/verify-ac/verify-ac.md` - Added JIRA prompt
- `.claude/commands/generate-e2e-tests/generate-e2e-tests.md` - Integrated selector scan, added platform selection
- `.claude/commands/collect-ac/collect-ac.md` - Renamed from start-task, updated workflow order
- `.claude/commands/setup-qa-assistant/setup-qa-assistant.md` - New command for dependency management

**Documentation:**
- `README.md` - Updated workflow, command references, documentation links
- `CHANGELOG.md` - Renamed from WORKFLOW-UPDATES-V2.1.md
- Moved supporting docs to `/docs` folder

**Unchanged:**
- `/post-to-jira` - Still available standalone
- `/smart-selector-scan` - Still available if needed
- `/figma-ac-extractor` - Optional enhancement
- `/ac-quality-trends` - Sprint reporting

---

**Last Updated: March 23, 2026**
