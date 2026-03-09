# Uncommitted Review Commands for OpenCode

A set of OpenCode commands for reviewing uncommitted git changes, providing structured, actionable feedback focusing on code quality, security, and performance.

## Overview

These commands review your uncommitted changes and provide structured, actionable feedback across multiple focus areas. The main command can delegate to specialized subcommands for deep security and performance analysis.

## Commands

| Command | Description |
|---------|-------------|
| `/uncommitted-review` | Main review command - analyzes all changes |
| `/security-review` | Specialized security-focused review |
| `/performance-review` | Specialized performance-focused review |

## Installation

1. **Clone or copy this repository** to your project:
   ```bash
   # Option 1: Clone to your project (recommended for per-project config)
   git clone <this-repo> .opencode

   # Option 2: Copy specific commands
   cp -r .opencode/command ~/.config/opencode/
   ```

2. **Restart OpenCode** to load the new commands

## Usage

### Main Review

```
/uncommitted-review
```

The main review command will:
1. Run `git diff HEAD --stat && git diff HEAD` to get all changes
2. Analyze the diff for general code quality issues
3. Optionally delegate to security/performance subcommands for deeper analysis
4. Save a complete review to `REVIEW-uncommitted-{timestamp}.md`

### Specialized Reviews

```
/security-review
```

Focused security analysis looking for:
- Input validation issues
- Authentication & authorization flaws
- Data exposure risks
- Cryptography issues

```
/performance-review
```

Focused performance analysis looking for:
- Algorithmic complexity issues
- Data structure & memory layout problems
- Allocation reduction opportunities
- Async/concurrency issues

### Before Commit Hook Integration

Add to `~/.gitconfig`:
```ini
[alias]
    review = !opencode run "/uncommitted-review"
```

Then run before committing:
```bash
git review
git commit
```

### Pre-commit Hook

Create `.git/hooks/pre-commit`:
```bash
#!/bin/bash
echo "Running uncommitted review..."

# Run OpenCode review
opencode run "/uncommitted-review" > review_output.md

# Check if review found critical/high issues
if grep -q "## Critical Issues\|## High Priority Issues" review_output.md; then
    echo ""
    echo "=== REVIEW FOUND BLOCKING ISSUES ==="
    cat review_output.md
    echo ""
    echo "Please address blocking issues before committing."
    exit 1
fi

echo "✓ No blocking issues found"
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

## Output

Reviews are saved to `REVIEW-uncommitted-{timestamp}.md` with the following structure:

```markdown
## Summary
Modified 3 files in auth module. Changes focus on JWT token validation and session management.

## Critical Issues
- **src/auth/jwt.py:42** - SQL Injection Vulnerability
  - Description: Direct string concatenation in query allows injection
  - Impact: Attackers could bypass authentication
  - Suggestion: Use parameterized queries

## High Priority Issues
- **src/auth/session.py:18** - Resource Leak
  - Description: Database connection not closed on exception
  - Impact: Connection pool exhaustion under load

## Medium Priority Issues
- **src/auth/validator.py:67** - Missing Type Hints
  - Description: Function returns int but has no annotation

## Low Priority Issues
- **src/auth/jwt.py:8** - Magic Number
  - Description: Hardcoded timeout value without explanation

## Suggestions
- Consider adding unit tests for the new validate_token function
- JWT secret should be in environment variables, not hardcoded

## Overall Assessment
- Quality Score: 6/10
- Ready to commit: No
- Required actions: Fix SQL injection vulnerability and resource leak
```

## Configuration

### Customize Focus Areas

Edit the command files to adjust focus areas:

```markdown
## Focus Areas

Prioritize issues in this order:

1. **Security Issues Only**:
   - Input validation and sanitization
   - Authentication and authorization checks
   - Secret/credential handling
   - SQL/query injection risks
```

### Adjust Temperature

Edit the frontmatter in the command file:
```markdown
---
description: Reviews uncommitted changes for code quality issues
agent: uncommitted-review
subtask: false
temperature: 0.1
---
```

### Modify Tool Permissions

The commands use the underlying agent. Edit the agent file in `.opencode/agents/` to adjust permissions:

```markdown
tools:
  bash:
    "git diff*": allow
    "git log*": allow
    "npm test": allow
    "*": deny
```

## Advanced Usage

### Review Against Specific Branch

```
/uncommitted-review Compare against develop branch, not main
```

### Focus on Specific Files

```
/uncommitted-review Only review src/auth/*.py files
```

### Ignore Test Files

```
/uncommitted-review Ignore all test files, only review source code
```

### Code Quality Gate for CI

In GitHub Actions:
```yaml
- name: Review uncommitted changes
  run: |
    opencode run "/uncommitted-review" > review.md
    
- name: Check for critical issues
  run: |
    if grep -q "## Critical Issues" review.md; then
      echo "Critical issues found, failing build"
      cat review.md
      exit 1
    fi
```

## Best Practices

### 1. Run Frequently
- Review after significant changes (not every small tweak)
- Review before pushing to remote
- Review before pull requests to catch issues early

### 2. Use Specialized Commands

- Run `/security-review` for auth, input handling, database code
- Run `/performance-review` for hot paths, data structures, algorithms
- Use both for significant changes covering both areas

### 3. Act on Feedback

- Fix Critical and High issues before committing
- Consider Medium issues (fix or document why not)
- Low issues are suggestions, not requirements

### 4. Use with Human Review

- This command is a tool, not a replacement for code review
- Use it to catch obvious issues before human review
- Combine with peer review for best results

## Troubleshooting

### Command not found

- Check files are in `.opencode/command/` or `~/.config/opencode/command/`
- Restart OpenCode after adding commands
- Verify filenames don't have special characters

### No changes detected

- Ensure you have uncommitted changes (`git status`)
- Check you're in a git repository
- Verify changes are tracked (not in `.gitignore`)

### Too much output

- Increase focus by using specialized commands
- Raise severity threshold in the command prompt
- Add file exclusions in your instructions

### False positives

- Add project-specific exceptions to the command prompt
- Use lower temperature for more deterministic results
- Document patterns the command should ignore

## Contributing

Found improvements? Consider:
1. Adjusting the prompts for better results
2. Adding new focus areas relevant to your stack
3. Improving the output format
4. Sharing your configuration with the community

## License

This agent configuration is adapted from:
- Qodo pr-agent (AGPL-3.0 License)
- Qodo agents repository (MIT License)

OpenCode agent configuration follows OpenCode's license.

## Related Resources

- [OpenCode Agents Documentation](https://opencode.ai/docs/agents/)
- [Qodo pr-agent GitHub](https://github.com/qodo-ai/pr-agent)
- [Qodo Agents GitHub](https://github.com/qodo-ai/agents)
- [OpenCode GitHub](https://github.com/anomalyco/opencode)
