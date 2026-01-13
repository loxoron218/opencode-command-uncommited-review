# Quick Installation & Usage Guide

## ✅ Agents Installed

Three OpenCode agents are now ready to use in `/home/arch/.config/opencode/agent/`:

1. **uncommitted-review** - General code quality review (security, performance, bugs, maintainability)
2. **security-review** - Security vulnerability analysis only
3. **performance-review** - Performance optimization opportunities only

## 🚀 Quick Start

### Step 1: Restart OpenCode

```bash
# Kill if running
pkill opencode

# Start fresh
opencode
```

### Step 2: Test Agents

In OpenCode, type one of these:

```
@uncommitted-review
```

```
@security-review
```

```
@performance-review
```

Press **Tab** after typing `@` to see autocomplete menu.

## 📋 Usage Examples

### Scenario 1: Review Before Commit

```bash
# Make changes
vim src/app.js

# Start OpenCode
opencode

# Review changes
@uncommitted-review

# Read feedback, fix issues
vim src/app.js

# Commit with confidence
git add .
git commit -m "Fix authentication issue"
```

### Scenario 2: Security Check on Auth Code

When working on sensitive code:

```
@security-review Focus on authentication and authorization
```

### Scenario 3: Performance Review After Optimization

After optimizing slow code:

```
@performance-review Check database query efficiency
```

## 🔧 Git Aliases (Optional)

Add to `~/.gitconfig`:

```ini
[alias]
    review = "!f() { opencode run \"@uncommitted-review\"; }; f"
    review-security = "!f() { opencode run \"@security-review\"; }; f"
    review-performance = "!f() { opencode run \"@performance-review\"; }; f"
```

Then use:
```bash
git review
git review-security
git review-performance
```

## 🛡️ Pre-commit Hook (Optional)

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
echo "🔍 Running code review..."

opencode run "@uncommitted-review" > /tmp/review.md

if grep -q "## Critical Issues\|## High Priority Issues" /tmp/review.md; then
    echo "❌ Blocking issues found!"
    cat /tmp/review.md
    exit 1
fi

echo "✅ Review passed!"
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

## 📊 Agent Comparison

| Agent | Focus | When to Use |
|-------|--------|-------------|
| **uncommitted-review** | Comprehensive (security, performance, bugs, style) | Everyday code reviews before commit |
| **security-review** | Security vulnerabilities only | Authentication, authorization, data handling code |
| **performance-review** | Performance issues only | After refactoring, optimizing slow paths |

## ⚙️ Configuration

All agents use:
- **Mode**: Subagent (invoked via @mention)
- **Temperature**: 0.1 (low randomness for consistent output)
- **Model**: Anthropic Claude Sonnet 4
- **Permissions**: Read-only (write/edit denied), bash limited to git commands

To modify, edit the agent files in `~/.config/opencode/agent/`

## ❓ Troubleshooting

### Agents not appearing

```bash
# Check files exist
ls -la ~/.config/opencode/agent/

# Should show:
# uncommitted-review.md
# security-review.md
# performance-review.md

# If not, copy them from the project:
cp *.md ~/.config/opencode/agent/

# Restart OpenCode
pkill opencode && opencode
```

### OpenCode won't start

```bash
# Move agents temporarily
mkdir -p ~/backup-agents
mv ~/.config/opencode/agent/*.md ~/backup-agents/

# Start OpenCode - should work
opencode

# Add back one at a time to isolate issue
cp ~/backup-agents/uncommitted-review.md ~/.config/opencode/agent/
```

### No review output

```bash
# Check you have uncommitted changes
git status
git diff --stat

# If no changes, make some:
echo "// TODO: test" >> src/app.js
```

### Too many false positives

Edit agent file to increase severity threshold:

Find this section and comment it out:
```markdown
## Low Priority Issues
# [Same format as Critical]
```

## 📚 Documentation

Full documentation available in project directory:

- **README.md** - Complete documentation with examples
- **QUICKSTART.md** - Detailed quick start guide
- **AGENT_INSTALLATION.md** - Installation notes and troubleshooting

## 🎓 Best Practices

1. **Run frequently** - Review after significant changes, not every tweak
2. **Fix critical issues** - Don't commit with Critical/High priority issues
3. **Use human review** - Agents catch obvious issues, humans catch nuanced ones
4. **Customize** - Edit prompts to match your team's standards
5. **Learn** - Note recurring issues to improve your coding patterns

## 🎉 Ready to Go!

You're all set! Start reviewing your code:

```bash
opencode
@uncommitted-review
```

Happy reviewing! 🚀
