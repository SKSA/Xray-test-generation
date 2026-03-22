# How to Share Your AC Workflow Commands with Colleagues

You have several options to share your custom AC workflow commands with teammates.

---

## 📦 Option 1: Share as a Zip File (Easiest)

### **Package the commands:**

```bash
cd /Users/mde/Documents/spectest
zip -r ac-workflow-commands.zip .claude/ *.md
```

This creates `ac-workflow-commands.zip` containing:
- `.claude/commands/` - All 7 custom commands
- All documentation files (README.md, WORKFLOW-UPDATES-V2.1.md, etc.)

### **Colleague installs:**

1. Extract the zip file
2. Copy `.claude/` folder to their project:
   ```bash
   cd ~/their-project
   unzip ac-workflow-commands.zip
   ```
3. Restart Claude Code
4. Commands appear: `/collect-ac`, `/verify-ac`, `/generate-e2e-tests`, etc.

---

## 🌐 Option 2: Git Repository (Best for Teams)

### **Create a shared repo:**

```bash
# In your spectest directory
cd /Users/mde/Documents/spectest
git init
git add .claude/ *.md
git commit -m "AC Workflow - Initial commit"

# Push to GitHub (internal repo)
gh repo create hellofresh/ac-workflow-internal --private
git remote add origin git@github.com:hellofresh/ac-workflow-internal.git
git push -u origin main
```

### **Colleagues clone and install:**

```bash
# Clone the workflow repo
git clone git@github.com:hellofresh/ac-workflow-internal.git

# Copy to their project
cd ~/their-project
cp -r ~/ac-workflow-internal/.claude .
```

### **Benefits:**
- ✅ Version control
- ✅ Easy updates (git pull)
- ✅ Team collaboration
- ✅ Track improvements

---

## 🔗 Option 3: Symlink (For Local Sharing)

If colleagues are on the same network/server:

```bash
# In colleague's project
cd ~/their-project
ln -s /Users/mde/Documents/spectest/.claude .claude
```

**Benefits:** Changes you make propagate automatically
**Drawback:** Only works on same machine/shared filesystem

---

## 📋 Option 4: Confluence/Google Drive Documentation

Share the markdown files for manual setup:

### **Upload to Confluence:**
1. Create page: "AC Workflow - Setup Guide"
2. Upload files:
   - `README.md` - Overview
   - `QUICK-START.md` - 5-minute guide
   - `AC-WORKFLOW-README.md` - Full guide
   - `WORKFLOW-UPDATES-V2.1.md` - Latest changes
3. Add command markdown files as attachments
4. Colleagues copy `.claude/commands/` structure manually

---

## 🎯 Option 5: spec-machine Plugin/Template (Future)

If spec-machine supports plugins, you could publish as:

```bash
# Hypothetical future command
spec-machine install ac-workflow-hellofresh
```

This would be ideal for company-wide sharing.

---

## ✅ Recommended Approach for HelloFresh

**For your team, I recommend:**

### **Approach: Git Repository + Documentation**

**Step 1: Create Internal Repo**
```bash
cd /Users/mde/Documents/spectest
git init
git add .claude/ *.md
git commit -m "feat: AC Verification Workflow v2.1

- Multi-source AC detection (JIRA, Confluence, Figma)
- Automated E2E test generation (Playwright, Cypress, Jest)
- Smart selector scanning
- JIRA integration (auto-post results)
- Quality trend tracking
- Framework selection
- Developer-friendly workflow

[Cursor_Code]"

# Create private repo
gh repo create hellofresh/ac-workflow --private --description "AC Verification Workflow for HelloFresh Engineering"
git remote add origin git@github.com:hellofresh/ac-workflow.git
git push -u origin main
```

**Step 2: Add README for Installation**
```bash
# Create INSTALL-COLLEAGUE.md
cat > INSTALL-COLLEAGUE.md << 'EOF'
# AC Workflow - Installation Guide

## Quick Install (5 minutes)

### 1. Clone the repo
git clone git@github.com:hellofresh/ac-workflow.git

### 2. Copy to your project
cd ~/your-project
cp -r ~/ac-workflow/.claude .

### 3. Restart Claude Code
Restart Claude Code to load new commands

### 4. Verify installation
In Claude chat, type:
/collect-ac

You should see the command available!

### 5. Install dependencies
- Install JIRA CLI: gh auth login (if not already)
- Install Figma MCP: /add-plugin figma
- Install Playwright: npm install -D @playwright/test

## Available Commands
- /collect-ac - Start with ACs
- /generate-e2e-tests - Generate tests
- /verify-ac - Verify ACs
- /post-to-jira - Post results
- /figma-ac-extractor - Get Figma ACs
- /smart-selector-scan - Scan selectors
- /ac-quality-trends - View metrics

## Full Documentation
See README.md for complete workflow guide
EOF

git add INSTALL-COLLEAGUE.md
git commit -m "docs: Add installation guide for colleagues"
git push
```

**Step 3: Share with Team**
```
Post in #engineering-tooling Slack:

🎯 New: AC Verification Workflow for Claude Code

Automates acceptance criteria verification with:
- Multi-source AC detection (JIRA + Confluence + Figma)
- Automated E2E test generation (Playwright, Cypress, Jest)
- Smart selector scanning
- JIRA integration (post results back)
- Quality tracking & metrics

📦 Install:
git clone git@github.com:hellofresh/ac-workflow.git
cd your-project
cp -r ~/ac-workflow/.claude .

📚 Full guide: https://github.com/hellofresh/ac-workflow

Questions? Ping me or check the README!
```

---

## 📊 What Gets Shared

### **Commands (`.claude/commands/`):**
1. `collect-ac/collect-ac.md` - Multi-source AC detection
2. `verify-ac/verify-ac.md` - Interactive AC verification
3. `generate-e2e-tests/generate-e2e-tests.md` - E2E test generation
4. `post-to-jira/post-to-jira.md` - JIRA integration
5. `smart-selector-scan/smart-selector-scan.md` - Selector scanning
6. `figma-ac-extractor/figma-ac-extractor.md` - Figma integration
7. `ac-quality-trends/ac-quality-trends.md` - Quality metrics

### **Documentation:**
- `README.md` - Main entry point
- `QUICK-START.md` - 5-minute guide
- `AC-WORKFLOW-README.md` - Complete guide
- `WORKFLOW-UPDATES-V2.1.md` - Latest changes
- `IMPROVEMENTS-V2.md` - Feature descriptions
- `FIGMA-AC-FOCUS.md` - Figma usage guide
- `INDEX.md` - Navigation

---

## 🔧 Colleague Setup Steps

Once they have the files:

### **1. Copy to Project**
```bash
cd ~/their-project
cp -r ~/ac-workflow/.claude .
```

### **2. Configure JIRA CLI**
```bash
# If not already configured
jira init
```

### **3. Install Figma MCP (Optional)**
```bash
# In Claude Code chat
/add-plugin figma
```

### **4. Test It**
```bash
# In Claude Code chat
/collect-ac EPS-1234
```

---

## 📦 Quick Package Script

Want to create a shareable package right now?

```bash
cd /Users/mde/Documents/spectest

# Create package
zip -r ac-workflow-v2.1.zip \
  .claude/ \
  README.md \
  QUICK-START.md \
  AC-WORKFLOW-README.md \
  WORKFLOW-UPDATES-V2.1.md \
  IMPROVEMENTS-V2.md \
  FIGMA-AC-FOCUS.md \
  INDEX.md

echo "✅ Created: ac-workflow-v2.1.zip"
echo "📧 Send to colleagues or upload to Confluence"
```

---

## 🎯 Next Steps

**For team-wide adoption:**

1. ✅ Create internal GitHub repo
2. ✅ Write installation guide
3. ✅ Share in Slack/email
4. ✅ Host demo session (15 min)
5. ✅ Create Confluence page
6. ✅ Track adoption metrics

**Demo Talking Points:**
- "Saves 2+ hours per ticket"
- "Automated test generation"
- "Multi-source AC detection"
- "Quality tracking built-in"
- "Works with existing JIRA workflow"

---

## 🤝 Support

**For colleagues who need help:**
- Slack: #ac-workflow-support (create channel)
- Docs: GitHub repo README
- Video: Record quick Loom demo
- Office Hours: Weekly 30min Q&A

---

## 📈 Track Adoption

Use quality trends to see team adoption:

```bash
/ac-quality-trends --team "Platform Engineering"
```

See who's using it, how often, and impact on velocity!

---

**Ready to share?** Pick Option 2 (Git Repo) for best results!
