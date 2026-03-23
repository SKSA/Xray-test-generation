# Dependency Management Guide

## For Teammates Using Your Commands

This guide explains how dependencies are managed when teammates start using the QA Assistant commands.

---

## 🎯 Automatic Dependency Detection

All commands **automatically check** for required dependencies before running. If something is missing, you'll be prompted to install it.

---

## 🚀 First-Time Setup

### **Step 1: Run Setup Command**

```bash
# In Claude Code chat
/setup-qa-assistant
```

**What it does:**
1. ✅ Checks all required tools (Git, Node.js, JIRA CLI, GitHub CLI)
2. ✅ Checks optional tools (Playwright, Maestro, Figma MCP)
3. ✅ Shows what's installed and what's missing
4. ✅ Offers to install missing dependencies
5. ✅ Configures tools after installation

**Example output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Dependency Check Results
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Required Dependencies (3/4)
  ✅ Git v2.39.0
  ✅ Node.js v20.10.0
  ✅ GitHub CLI v2.40.0
  ❌ JIRA CLI - Not installed

⚠️ Optional Dependencies (1/3)
  ❌ Playwright - Not installed
  ❌ Maestro - Not installed
  ✅ Figma MCP - Configured

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Would you like me to install missing dependencies? (Y/n)
```

### **Step 2: Accept Installation**

```
You: y

Installing JIRA CLI...
✅ JIRA CLI installed

Installing Playwright...
✅ Playwright installed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 Installation Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next steps:
1. Configure JIRA: jira init
2. Authenticate GitHub: gh auth login
3. Try the workflow: /collect-ac EPS-1234
```

---

## 📦 Required Dependencies

### **Always Needed:**

| Tool | Purpose | Auto-Installed |
|------|---------|----------------|
| **Git** | Version control | ❌ (Usually pre-installed) |
| **Node.js 18+** | Test frameworks | ❌ (Manual install) |
| **JIRA CLI** | Fetch tickets | ✅ Yes |
| **GitHub CLI** | Create PRs | ✅ Yes |

### **Platform-Specific:**

**For Web Testing:**
- **Playwright** or **Cypress** - Auto-installed when you choose Web platform

**For React Native Testing:**
- **Maestro** or **Detox** - Auto-installed when you choose Mobile platform

**For Figma Integration:**
- **Figma MCP** - Auto-configured if you use `/figma-ac-extractor`

---

## 🔄 Per-Command Dependency Checks

Each command checks its own dependencies before running:

### **/collect-ac**

**Checks:**
- ✅ JIRA CLI (if fetching from JIRA)
- ✅ Confluence API access (if fetching from Confluence)

**Prompt if missing:**
```
⚠️ JIRA CLI not found

The /collect-ac command needs JIRA CLI to fetch tickets.

Install now? (Y/n)

→ Installing JIRA CLI...
→ ✅ Installed! Run 'jira init' to configure.
```

### **/generate-e2e-tests**

**Checks based on platform:**

**Web:**
- ✅ Playwright or Cypress

**Mobile React Native:**
- ✅ Maestro or Detox
- ✅ React Native Testing Library

**Prompt if missing:**
```
⚠️ No E2E framework detected

For Web testing, you need:
  • Playwright (recommended)
  • or Cypress

Install Playwright? (Y/n)

→ Running: npm install -D @playwright/test
→ Running: npx playwright install
→ ✅ Playwright ready!
```

### **/figma-ac-extractor**

**Checks:**
- ✅ Figma MCP configured

**Prompt if missing:**
```
⚠️ Figma MCP not configured

To extract ACs from Figma, you need the Figma MCP server.

Configure now? (Y/n)

→ Running: claude mcp add figma https://mcp.figma.com/mcp
→ ✅ Configured! Restart Claude Code and run: /mcp
```

### **/verify-ac**

**Checks:**
- ✅ Test runner exists (Playwright/Maestro/Jest)

**Prompt if missing:**
```
⚠️ No test results found

Run your tests first:
  Web: npx playwright test
  Mobile: maestro test .maestro/

Then run /verify-ac again.
```

---

## 🛠️ Manual Installation

If automatic installation fails, here's how to install manually:

### **JIRA CLI**

**macOS:**
```bash
brew install ankitpokhrel/jira-cli/jira-cli
jira init
```

**Linux:**
```bash
curl -sL https://github.com/ankitpokhrel/jira-cli/releases/latest/download/jira_linux_x86_64.tar.gz | tar xz
sudo mv jira /usr/local/bin/
jira init
```

**Windows:**
```bash
scoop install jira-cli
jira init
```

### **GitHub CLI**

**macOS:**
```bash
brew install gh
gh auth login
```

**Linux:**
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo apt-get install gh
gh auth login
```

**Windows:**
```bash
scoop install gh
gh auth login
```

### **Playwright**

```bash
npm install -D @playwright/test
npx playwright install
```

### **Maestro**

```bash
curl -Ls "https://get.maestro.mobile.dev" | bash
export PATH=$PATH:$HOME/.maestro/bin
```

### **Figma MCP**

```bash
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
# Restart Claude Code
# In Claude chat: /mcp → Select figma → Authenticate
```

---

## ⚡ Quick Start for Teammates

**Send this to your colleagues:**

```
🚀 Quick Start: QA Assistant

1. Clone repo:
   git clone git@github.com:mde-hf/spec-dev-qa-assistant.git
   cd your-project
   cp -r ~/spec-dev-qa-assistant/.claude .

2. Restart Claude Code

3. Run setup (checks & installs dependencies):
   /setup-qa-assistant

4. Follow prompts to install missing tools

5. Configure JIRA (if needed):
   jira init

6. Try it:
   /collect-ac EPS-1234

That's it! All commands will check dependencies automatically.
```

---

## 🔍 Troubleshooting

### **Issue: Command says dependency missing but it's installed**

```bash
# Re-run setup to refresh detection
/setup-qa-assistant

# Or check manually
which jira
which gh
which maestro
```

### **Issue: Permission denied during installation**

```bash
# macOS: Install Homebrew first
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Linux: Use sudo for system-wide installs
sudo apt-get install ...
```

### **Issue: Network errors during installation**

- Check internet connection
- Check if behind corporate proxy
- Try manual installation from official sites

### **Issue: Playwright browsers not installing**

```bash
# Run manually
npx playwright install

# Or specific browser
npx playwright install chromium
```

---

## 📊 Dependency Status Check

**At any time, check what's installed:**

```bash
/setup-qa-assistant
```

**Or check manually:**

```bash
# Check all core dependencies
git --version
node --version
jira --version
gh --version

# Check test frameworks
npx playwright --version
maestro --version

# Check Figma MCP
cat ~/.claude.json | grep figma
```

---

## 🎯 Platform-Specific Dependencies

### **Web Development**

**Required:**
- Node.js 18+
- Playwright or Cypress

**Optional:**
- Jest (unit tests)
- React Testing Library (component tests)
- Supertest (API tests)

### **React Native Development**

**Required:**
- Node.js 18+
- React Native environment
- Maestro or Detox

**Optional:**
- React Native Testing Library
- Jest
- iOS Simulator / Android Emulator

---

## 💾 Configuration File

After setup, a config file is created:

**Location:** `.qa-assistant-config.json`

**Contents:**
```json
{
  "setupComplete": true,
  "dependencies": {
    "jira-cli": "2.1.0",
    "gh": "2.40.0",
    "playwright": "1.40.0",
    "figma-mcp": "configured"
  },
  "setupDate": "2026-03-22T10:30:00Z",
  "platform": "Darwin"
}
```

**This file helps:**
- Skip dependency checks if already installed
- Track what's configured
- Debug setup issues

---

## 🆘 Getting Help

**If dependencies won't install:**

1. Check your platform (macOS/Linux/Windows)
2. Check Node.js version (must be 18+)
3. Try manual installation
4. Check network/proxy settings
5. Ask in #spec-dev-qa-assistant Slack channel

---

**Key Point:** Teammates don't need to know what dependencies are required upfront. Just run `/setup-qa-assistant` and follow prompts! 🚀
