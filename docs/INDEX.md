# X-Ray Test Generator - Documentation Index

## 📁 Location
**Base Directory:** `/Users/sku/Desktop/spec-dev-qa-assistant/`

---

## 📚 Documentation

### Main Documentation
- **[GENERATE-XRAY-TESTS-GUIDE.md](./GENERATE-XRAY-TESTS-GUIDE.md)** - Complete X-Ray integration guide (18KB)
  - Interactive format selection
  - Performance optimizations
  - JIRA and X-Ray API integration
  - Comprehensive troubleshooting
  - Best practices and examples

### Quick Reference
- **[README.md](../README.md)** - Project overview and quick start (8KB)
  - Command reference
  - Basic workflow
  - Prerequisites and setup

---

## ⚙️ Command File

Located in: `.claude/commands/`

### X-Ray Test Generation Command
**File:** `.claude/commands/generate-xray-tests/command.md`

**Purpose:** Generate X-Ray test cases from JIRA ticket acceptance criteria

**Usage:** `/generate-xray-tests TICKET-ID [--format=bdd|manual] [--dry-run]`

**Creates:**
- X-Ray test cases directly in JIRA (no local files)
- Issue links between tests and original ticket

---

## 🚀 Quick Start

```bash
# 1. Setup environment
export JIRA_API_TOKEN="your-api-token"
export JIRA_URL="https://company.atlassian.net"

# 2. Copy command to your repo
cp -r .claude/commands your-repo/.claude/

# 3. Use in Claude Code
/generate-xray-tests PROJ-123
```

---

## 🎯 Workflow Summary

```
┌──────────────────────────────────────────────────┐
│                                                  │
│  1. /generate-xray-tests TICKET                 │
│     → Extracts ACs from JIRA                    │
│     → Interactive format selection              │
│     → Creates X-Ray test cases                  │
│     → Links to original ticket                  │
│                                                  │
│  2. View results in JIRA                        │
│     → Navigate to ticket                        │
│     → See linked X-Ray test cases               │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 📂 Directory Structure

```
/Users/sku/Desktop/spec-dev-qa-assistant/
│
├── 📖 Documentation
│   ├── README.md                       # Project overview
│   ├── CHANGELOG.md                    # Version history
│   └── docs/
│       ├── INDEX.md                    # This file
│       └── GENERATE-XRAY-TESTS-GUIDE.md # Complete guide
│
├── ⚙️ Command
│   └── .claude/commands/
│       └── generate-xray-tests/
│           └── command.md
│
└── 📋 Configuration
    └── package.json
```

---

## 🔗 Related Resources

- **JIRA CLI:** [ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli)
- **X-Ray Documentation:** [Atlassian X-Ray](https://www.atlassian.com/software/jira/features/test-management)
- **JIRA REST API:** [Atlassian Developer](https://developer.atlassian.com/server/jira/platform/rest-apis/)

---

## 📞 Getting Help

**Questions about:**
- **Basic usage:** See `README.md`
- **Detailed workflow:** See `GENERATE-XRAY-TESTS-GUIDE.md`
- **Command options:** See `.claude/commands/generate-xray-tests/command.md`
- **Troubleshooting:** See troubleshooting section in the guide

---

## ✅ Next Steps

1. **Read the overview:**
   ```bash
   cat README.md
   ```

2. **Read the complete guide:**
   ```bash
   cat docs/GENERATE-XRAY-TESTS-GUIDE.md
   ```

3. **Try it in your repo:**
   ```bash
   cp -r .claude/commands your-repo/.claude/
   cd your-repo
   /generate-xray-tests YOUR-TICKET
   ```

---

**Created:** April 7, 2026
**Location:** `/Users/sku/Desktop/spec-dev-qa-assistant/`
**Purpose:** X-Ray test case generation from JIRA acceptance criteria