# Contributing to X-Ray Test Generator

Thank you for contributing to X-Ray Test Generator! This guide will help you get started.

## Quick Links

- 📚 [X-Ray Integration Guide](docs/GENERATE-XRAY-TESTS-GUIDE.md) - Complete usage guide
- 🐛 [Issue Tracker](https://github.com/SKSA/Xray-test-generation/issues) - Report bugs or request features

## Your First Contribution

**Choose your path based on what you want to do:**

| What you want to do | Time | Steps |
|---------------------|------|-------|
| Fix a typo or improve docs | 10 min | Edit → Commit → PR |
| Add an example | 20 min | Find file → Add example → PR |
| Report a bug | 15 min | [Create issue](https://github.com/SKSA/Xray-test-generation/issues/new) with repro steps |
| Fix a bug | 1-2 hrs | Setup → Fix → Test → PR |
| Add a new feature | 2-4 hrs | [Start a discussion](https://github.com/SKSA/Xray-test-generation/discussions) first |

### Quick Start: I Just Want to Fix a Typo

**No setup needed! Just use GitHub's web interface:**

1. Browse to the file with the typo
2. Click the edit (pencil) icon
3. Make your change
4. Scroll down → "Propose changes"
5. Click "Create pull request"

Done! We'll review and merge quickly.

### 1. Find Something to Work On

**Good first contributions:**
- Fix typos or improve documentation
- Add examples to documentation
- Report bugs with clear reproduction steps
- Suggest new features in discussions

Browse [existing issues](https://github.com/SKSA/Xray-test-generation/issues) or [create a new one](https://github.com/SKSA/Xray-test-generation/issues/new).

### 2. Set Up Your Environment

**Prerequisites:**
- Node.js 18+ or Bun
- [GitHub CLI](https://cli.github.com/) (optional)
- Git
- [JIRA CLI](https://github.com/ankitpokhrel/jira-cli) for testing

**Clone and install:**
```bash
gh repo clone SKSA/Xray-test-generation
# or
git clone https://github.com/SKSA/Xray-test-generation.git

cd Xray-test-generation
npm install
# or
bun install
```

**Verify setup:**
```bash
npm test      # Run all tests
npm run lint  # Check code style
```

### 3. Make Your Changes

**Branch naming:**
```bash
git checkout -b fix/description       # Bug fixes
git checkout -b feat/description      # New features
git checkout -b docs/description      # Documentation
```

**Before committing:**
```bash
npm run lint       # Check for lint issues
npm run format     # Format code (if available)
npm test           # All tests pass
```

### 4. Submit a Pull Request

1. **Push your branch:** `git push origin your-branch-name`
2. **Open a pull request on GitHub**
3. **Fill out the PR template** (describes what changed and why)
4. **Wait for review** - we'll provide feedback and may request changes

**PR Requirements:**
- All tests must pass
- Code follows project style guidelines
- Documentation is updated if needed
- Commit messages are clear and descriptive

## Types of Contributions

### 🐛 Bug Fixes

1. [Create an issue](https://github.com/SKSA/Xray-test-generation/issues/new) describing the bug
2. Reference the issue in your PR: "Fixes #123"
3. Include test cases that verify the fix (if applicable)

### ✨ New Features

1. [Start a discussion](https://github.com/SKSA/Xray-test-generation/discussions) first
2. Get feedback on the approach
3. Implement with tests and documentation
4. Submit PR referencing the discussion

### 📚 Documentation

Documentation improvements are always welcome! No need for discussion first.

Areas to improve:
- Fix typos or unclear explanations
- Add missing examples
- Improve getting started guides
- Document undocumented features

## Development Workflow

### Testing Locally

**Test the command in your project:**
```bash
# Copy to your project
cp -r .claude ~/your-project/

# In Claude Code chat
/generate-xray-tests TICKET-123
```

### Code Quality

**Style guidelines:**
- Clear, descriptive variable names
- Comments for complex logic only
- Follow existing patterns in the codebase
- Keep functions focused and single-purpose

## Project Structure

```
Xray-test-generation/
├── .claude/
│   └── commands/
│       └── generate-xray-tests/
│           └── command.md         # Main command implementation
├── docs/                          # Documentation
│   ├── GENERATE-XRAY-TESTS-GUIDE.md
│   ├── QUICK-START.md
│   └── INDEX.md
├── CHANGELOG.md                   # Version history
├── README.md                      # Project overview
├── CONTRIBUTING.md                # This file
└── package.json                   # Project metadata
```

## Getting Help

- 💬 [Discussions](https://github.com/SKSA/Xray-test-generation/discussions) - Ask questions
- 🐛 [Issues](https://github.com/SKSA/Xray-test-generation/issues) - Report bugs
- 📖 [Documentation](docs/) - Read guides

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what's best for the project
- Welcome newcomers and help them learn

## License

By contributing, you agree that your contributions will be licensed under the same license as the project (MIT).

---

**Still have questions?** Check out our [detailed documentation](docs/) or [ask in discussions](https://github.com/SKSA/Xray-test-generation/discussions).
