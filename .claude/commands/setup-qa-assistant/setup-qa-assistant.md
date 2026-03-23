---
name: /setup-qa-assistant
argument-hint: ''
description: Check and install dependencies for QA workflow
mode: single-agent
---

# Setup QA Assistant

## Purpose
Check for required dependencies and guide installation of missing tools.

## Workflow

### Step 1: Welcome

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔧 QA Assistant Dependency Check
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Checking your machine for required dependencies...
```

### Step 2: Check Core Dependencies

**A. JIRA CLI (Required for JIRA integration)**

```bash
# Check if JIRA CLI is installed
which jira

if [ $? -eq 0 ]; then
  jira --version
  echo "✅ JIRA CLI installed"
else
  echo "❌ JIRA CLI not found"
  MISSING_DEPS+=("jira-cli")
fi
```

**B. Git (Required)**

```bash
# Check git
git --version

if [ $? -eq 0 ]; then
  echo "✅ Git installed"
else
  echo "❌ Git not found"
  MISSING_DEPS+=("git")
fi
```

**C. GitHub CLI (Required for PR creation)**

```bash
# Check gh CLI
gh --version

if [ $? -eq 0 ]; then
  echo "✅ GitHub CLI installed"
  gh auth status
else
  echo "❌ GitHub CLI not found"
  MISSING_DEPS+=("gh")
fi
```

**D. Node.js (Required for test frameworks)**

```bash
# Check Node.js
node --version

if [ $? -eq 0 ]; then
  NODE_VERSION=$(node --version)
  echo "✅ Node.js installed ($NODE_VERSION)"
  
  # Check if version is >= 18
  MAJOR_VERSION=$(echo $NODE_VERSION | cut -d. -f1 | sed 's/v//')
  if [ $MAJOR_VERSION -lt 18 ]; then
    echo "⚠️ Node.js 18+ recommended (current: $NODE_VERSION)"
  fi
else
  echo "❌ Node.js not found"
  MISSING_DEPS+=("node")
fi
```

### Step 3: Check Optional Dependencies

**A. Playwright (for Web E2E tests)**

```bash
# Check if Playwright is installed in project
if [ -f "package.json" ]; then
  if grep -q "@playwright/test" package.json; then
    echo "✅ Playwright installed in project"
  else
    echo "⚠️ Playwright not installed (optional for Web E2E)"
    OPTIONAL_DEPS+=("playwright")
  fi
fi
```

**B. Maestro (for React Native E2E tests)**

```bash
# Check if Maestro is installed
which maestro

if [ $? -eq 0 ]; then
  maestro --version
  echo "✅ Maestro installed"
else
  echo "⚠️ Maestro not installed (optional for React Native)"
  OPTIONAL_DEPS+=("maestro")
fi
```

**C. Detox (for React Native E2E tests)**

```bash
# Check if Detox is installed
if [ -f "package.json" ]; then
  if grep -q "detox" package.json; then
    echo "✅ Detox installed in project"
  else
    echo "⚠️ Detox not installed (optional for React Native)"
  fi
fi
```

**D. Figma MCP (for Figma integration)**

```bash
# Check if Figma MCP is configured
if [ -f "$HOME/.claude.json" ]; then
  if grep -q "figma" "$HOME/.claude.json"; then
    echo "✅ Figma MCP configured"
  else
    echo "⚠️ Figma MCP not configured (optional)"
    OPTIONAL_DEPS+=("figma-mcp")
  fi
fi
```

### Step 4: Display Results

Show summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Dependency Check Results
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Required Dependencies (4/4)
  ✅ Git
  ✅ Node.js
  ✅ JIRA CLI
  ✅ GitHub CLI

⚠️ Optional Dependencies (2/4)
  ✅ Playwright
  ✅ Figma MCP
  ❌ Maestro
  ❌ Detox
```

### Step 5: Offer Installation

If missing dependencies found, ask:

```
❌ Missing Required Dependencies:
  - JIRA CLI
  - GitHub CLI

Would you like me to install these now? (Y/n)
```

If user says **Yes**, proceed with installations:

### Step 6: Install Missing Dependencies

**A. JIRA CLI**

```bash
echo "📦 Installing JIRA CLI..."

# Detect OS
if [[ "$OSTYPE" == "darwin"* ]]; then
  # macOS
  brew install ankitpokhrel/jira-cli/jira-cli
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  # Linux
  curl -sL https://github.com/ankitpokhrel/jira-cli/releases/latest/download/jira_linux_x86_64.tar.gz | tar xz
  sudo mv jira /usr/local/bin/
fi

echo "✅ JIRA CLI installed"
echo "Run 'jira init' to configure your JIRA instance"
```

**B. GitHub CLI**

```bash
echo "📦 Installing GitHub CLI..."

# Detect OS
if [[ "$OSTYPE" == "darwin"* ]]; then
  # macOS
  brew install gh
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  # Linux
  curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
  sudo apt-get update
  sudo apt-get install gh
fi

echo "✅ GitHub CLI installed"
echo "Run 'gh auth login' to authenticate"
```

**C. Playwright (if user wants)**

```bash
echo "📦 Installing Playwright..."
npm install -D @playwright/test
npx playwright install

echo "✅ Playwright installed"
```

**D. Maestro (if user wants)**

```bash
echo "📦 Installing Maestro..."
curl -Ls "https://get.maestro.mobile.dev" | bash

echo "✅ Maestro installed"
echo "Add to PATH: export PATH=$PATH:$HOME/.maestro/bin"
```

**E. Figma MCP**

```bash
echo "📦 Configuring Figma MCP..."

# Run the setup command
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp

echo "✅ Figma MCP configured"
echo "Restart Claude Code and authenticate: /mcp"
```

### Step 7: Configuration Guidance

After installation, provide setup instructions:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 Installation Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 Next Steps:

1. Configure JIRA CLI:
   jira init

2. Authenticate GitHub CLI:
   gh auth login

3. (Optional) Authenticate Figma MCP:
   In Claude Code: /mcp → Select figma → Authenticate

4. Test the workflow:
   /collect-ac EPS-1234

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ You're ready to use the QA Assistant!
```

### Step 8: Save Configuration

Create a local config file to track setup:

```bash
# Create .qa-assistant-config
cat > .qa-assistant-config.json << EOF
{
  "setupComplete": true,
  "dependencies": {
    "jira-cli": "installed",
    "gh": "installed",
    "playwright": "installed",
    "figma-mcp": "configured"
  },
  "setupDate": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")",
  "platform": "$(uname)"
}
EOF

echo "✅ Configuration saved to .qa-assistant-config.json"
```

## Error Handling

| Issue | Solution |
|-------|----------|
| Homebrew not found (macOS) | Provide manual installation instructions |
| Permission denied | Suggest using sudo or checking permissions |
| Network issues | Suggest checking internet connection |
| Already installed | Skip and show current version |

## Output

Configuration file: `.qa-assistant-config.json`

## Usage

Run this command:
- First time using QA Assistant
- After cloning repo on new machine
- When dependencies seem broken
- To check what's installed
