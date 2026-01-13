# Quick Start Guide: Uncommitted Review Agent

Get started with reviewing your uncommitted changes in 5 minutes.

## Installation

### Step 1: Copy the Agent

```bash
# Create OpenCode agent directory if needed
mkdir -p ~/.config/opencode/agent

# Copy the review agent
cp opencode-agent-uncommitted-review.md ~/.config/opencode/agent/
```

### Step 2: (Optional) Copy Specialized Agents

```bash
# Security-focused review
cp opencode-agent-security-review.md ~/.config/opencode/agent/

# Performance-focused review
cp opencode-agent-performance-review.md ~/.config/opencode/agent/
```

### Step 3: Restart OpenCode

Close and reopen OpenCode to load the new agents.

## Basic Usage

### Scenario 1: Review Before Committing

1. Make some changes:
   ```bash
   echo "// TODO: fix this" >> src/app.js
   ```

2. Start OpenCode:
   ```bash
   opencode
   ```

3. Type in the OpenCode prompt:
   ```
   @uncommitted-review
   ```

4. Read the feedback and fix issues

5. Commit with confidence:
   ```bash
   git commit -m "Fixed TODOs after review"
   ```

### Scenario 2: Security-Focused Review

When working on auth/permissions:

```
@security-review Focus on authentication code
```

### Scenario 3: Performance Review

When optimizing slow code:

```
@performance-review Check the database query optimization
```

## Git Alias Integration

Add this to `~/.gitconfig`:

```ini
[alias]
    review = "!f() { opencode run \"@uncommitted-review\"; }; f"
    review-security = "!f() { opencode run \"@security-review\"; }; f"
    review-performance = "!f() { opencode run \"@performance-review\"; }; f"
```

Now you can:

```bash
# General review
git review

# Security review
git review-security

# Performance review
git review-performance
```

## Pre-commit Hook (Automated)

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash

echo "🔍 Running code review..."

# Run review and save output
opencode run "@uncommitted-review" > /tmp/review.md

# Check for critical issues
if grep -q "## Critical Issues\|## High Priority Issues" /tmp/review.md; then
    echo ""
    echo "❌ BLOCKING ISSUES FOUND"
    echo "==================================="
    cat /tmp/review.md
    echo ""
    echo "Please fix blocking issues before committing."
    exit 1
fi

# Check for security agent if available
if grep -q "## Critical Vulnerabilities" /tmp/review.md 2>/dev/null; then
    echo "❌ SECURITY VULNERABILITIES FOUND"
    echo "==================================="
    cat /tmp/review.md
    echo ""
    echo "Please fix security issues immediately!"
    exit 1
fi

echo "✅ Review passed! No blocking issues."
exit 0
```

Make it executable:

```bash
chmod +x .git/hooks/pre-commit
```

Now every commit will be automatically reviewed!

## CI/CD Integration

### GitHub Actions

Create `.github/workflows/review.yml`:

```yaml
name: Code Review

on:
  push:
    branches: [main, develop]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install OpenCode
        run: npm install -g @opencode/cli

      - name: Run Security Review
        run: |
          opencode run "@security-review" > security-review.md

      - name: Check for Security Issues
        run: |
          if grep -q "## Critical Vulnerabilities" security-review.md; then
            echo "🚨 Security vulnerabilities found!"
            cat security-review.md
            exit 1
          fi

      - name: Run Performance Review
        run: |
          opencode run "@performance-review" > performance-review.md

      - name: Upload Reviews
        uses: actions/upload-artifact@v4
        with:
          name: code-reviews
          path: "*.md
```

## Common Workflows

### Before Pull Request

```bash
# Make changes
vim src/auth.js

# Review them
git review

# Fix issues
vim src/auth.js

# Stage and commit
git add .
git commit -m "Add auth middleware"

# Push and create PR
git push origin feature/auth-middleware
```

### After Refactoring

```bash
# Refactor code
vim src/utils.js

# Performance review
git review-performance

# Security check
git review-security

# General review
git review

# Commit if passed
git commit -m "Refactor utils.js"
```

### Bug Fix

```bash
# Fix the bug
vim src/api.js

# Quick review (check you didn't introduce new issues)
git review

# Commit
git commit -m "Fix user authentication bug"
```

## Customization Tips

### Focus on Specific Files

```
@uncommitted-review Only review src/auth/ directory
```

### Exclude Files

```
@uncommitted-review Ignore test files, focus on source code
```

### Change Severity Threshold

Edit the agent file, find this section:

```markdown
3. **Medium Priority**:
```

Change to:

```markdown
## Medium Priority Issues (skip this section)
# Only report Critical and High issues
```

### Add Project-Specific Checks

Add to the agent's "What to Check" section:

```markdown
### My Project Specific
- Are you using our custom utility functions?
- Are error messages in the correct format?
- Are logging statements present at critical points?
```

## Troubleshooting

### Agent not found

**Problem**: `@uncommitted-review` doesn't autocomplete

**Solution**:
1. Check file exists: `ls ~/.config/opencode/agent/`
2. Restart OpenCode completely
3. Check filename is correct (no typos)

### No output

**Problem**: Review completes instantly with no feedback

**Solution**:
1. Check you have uncommitted changes: `git status`
2. Verify you're in a git repository: `git branch`
3. Try explicit command: `git diff` to see changes

### Too many false positives

**Problem**: Review reports minor issues

**Solution**:
1. Edit agent file, increase severity threshold
2. Add project-specific exceptions to the prompt
3. Use `temperature: 0.0` for more consistent results

### Hook blocks everything

**Problem**: Pre-commit hook always fails

**Solution**:
1. Check hook is executable: `ls -la .git/hooks/pre-commit`
2. Test manually: `.git/hooks/pre-commit`
3. Adjust what it blocks in the script

## Next Steps

1. **Customize**: Edit agent files to match your team's standards
2. **Integrate**: Set up pre-commit hooks and CI/CD
3. **Share**: Teach your team how to use the agents
4. **Iterate**: Adjust prompts based on feedback
5. **Extend**: Create your own specialized agents

## Examples

See the `examples/` directory for:
- Different agent configurations
- CI/CD setups
- Git hook examples
- Team workflows

## Support

- [OpenCode Documentation](https://opencode.ai/docs/)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
- [OpenCode Discord](https://opencode.ai/discord)

Happy reviewing! 🎉
