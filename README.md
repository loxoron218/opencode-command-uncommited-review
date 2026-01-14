# Uncommitted Review Agent for OpenCode

A specialized OpenCode subagent for reviewing uncommitted git changes, adapted from Qodo's PR review methodology.

## Overview

This agent reviews your uncommitted changes against a target branch (default: main) and provides structured, actionable feedback focusing on code quality, security, performance, and maintainability.

**Inspired by:**
- Qodo's `pr-agent` - LocalGitProvider architecture
- Qodo's agents repository - Structured output and configuration
- OpenCode's Plan agent - Read-only analysis with permissions

## Key Features

### From Qodo pr-agent
- **Structured severity levels**: Critical, High, Medium, Low
- **Line-specific feedback**: References exact file:line from diffs
- **Comprehensive focus areas**: Security, Performance, Correctness, Maintainability
- **Actionable output**: Clear suggestions with code examples
- **Quality scoring**: X/10 assessment with commit readiness

### From Qodo agents
- **Customizable configuration**: Adjust via OpenCode's agent system
- **Structured output format**: Consistent markdown structure
- **Flexible severity thresholds**: Focus on what matters to you

### OpenCode Integration
- **Read-only permissions**: Can't accidentally modify your code
- **Bash access control**: Only git commands allowed
- **Subagent mode**: Invoke via `@uncommitted-review` or automatically
- **Temperature optimized**: Low randomness for consistent reviews

## Installation

1. **Create agent directory** (if needed):
   ```bash
   mkdir -p ~/.config/opencode/agent
   ```

2. **Copy agent file**:
   ```bash
   cp opencode-agent-uncommitted-review.md ~/.config/opencode/agent/
   ```

3. **Restart OpenCode** to load the new agent

## Usage

### Method 1: Manual Invocation

In OpenCode TUI, type:
```
@uncommitted-review
```

The agent will:
1. Run `git diff --stat` to see what changed
2. Run `git diff` to analyze the changes
3. Provide structured feedback

### Method 2: Before Commit Hook Integration

Add to `~/.gitconfig`:
```ini
[alias]
    review = !opencode run \"@uncommitted-review\"
```

Then run before committing:
```bash
git review
git commit
```

### Method 3: Pre-commit Hook

Create `.git/hooks/pre-commit`:
```bash
#!/bin/bash
echo "Running uncommitted review..."

# Run OpenCode review
opencode run "@uncommitted-review" > review_output.md

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

## Output Example

```markdown
## Summary
Modified 3 files in auth module. Changes focus on JWT token validation and session management.

## Critical Issues
- **src/auth/jwt.py:42** - SQL Injection Vulnerability
  - Description: Direct string concatenation in query allows injection
  - Impact: Attackers could bypass authentication
  - Suggestion: Use parameterized queries
  - Code Example:
    ```python
    # Bad:
    query = f"SELECT * FROM users WHERE token = '{token}'"
    
    # Good:
    query = "SELECT * FROM users WHERE token = ?"
    cursor.execute(query, (token,))
    ```

## High Priority Issues
- **src/auth/session.py:18** - Resource Leak
  - Description: Database connection not closed on exception
  - Impact: Connection pool exhaustion under load
  - Suggestion: Use context manager
  - Code Example:
    ```python
    # Good:
    with db.get_connection() as conn:
        conn.execute(...)
    ```

## Medium Priority Issues
- **src/auth/validator.py:67** - Missing Type Hints
  - Description: Function returns int but has no annotation
  - Impact: Reduced IDE support and type checking
  - Suggestion: Add -> int return type

## Low Priority Issues
- **src/auth/jwt.py:8** - Magic Number
  - Description: Hardcoded timeout value without explanation
  - Impact: Maintenance difficulty
  - Suggestion: Extract to named constant

## Suggestions
- Consider adding unit tests for the new validate_token function
- JWT secret should be in environment variables, not hardcoded
- Session cleanup job might need error handling

## Overall Assessment
- Quality Score: 6/10
- Ready to commit: No
- Required actions: Fix SQL injection vulnerability and resource leak
```

## Configuration

### Adjust Severity Focus

Edit the agent file to change focus areas or severity definitions. For example, to focus only on security:

```markdown
## Focus Areas

Prioritize issues in this order:

1. **Security Issues Only**:
   - Input validation and sanitization
   - Authentication and authorization checks
   - Secret/credential handling
   - SQL/query injection risks
   - XSS vulnerabilities

[...skip other categories...]
```

### Change Temperature

Edit the frontmatter to adjust response randomness:
```markdown
temperature: 0.1  # More deterministic (0.0-0.2)
temperature: 0.3  # Balanced (0.3-0.5)
temperature: 0.7  # More creative (0.6-1.0)
```

### Modify Tool Permissions

Allow additional bash commands if needed:
```markdown
tools:
  bash:
    "git diff*": allow
    "git log*": allow
    "npm test": allow  # Add this
    "*": deny
```

### Change Model

Use a different model:
```markdown
model: openai/gpt-5-codex
# or
model: anthropic/claude-haiku-4-20250514
```

## Differences from Qodo pr-agent

| Aspect | Qodo pr-agent | OpenCode Uncommitted Review |
|--------|---------------|---------------------------|
| **Context** | Pull Requests | Uncommitted changes |
| **Provider** | GitHub/GitLab/etc + Local | Local only |
| **Diff Source** | PR diff API | `git diff` |
| **Output Format** | YAML | Markdown |
| **Invocation** | `/review` command | `@uncommitted-review` subagent |
| **Integration** | GitHub Actions, webhooks | Pre-commit hooks, CLI |
| **Permissions** | Write to PR comments | Read-only (Plan mode) |

## Differences from Qodo agents

| Aspect | Qodo agents | OpenCode Uncommitted Review |
|--------|-------------|---------------------------|
| **Config Format** | TOML/YAML | Markdown with frontmatter |
| **Tool System** | MCP servers | OpenCode tools |
| **Execution** | Qodo CLI | OpenCode TUI/CLI |
| **Output Schema** | JSON with validation | Markdown structure |
| **Exit Expressions** | CI/CD blocking | Manual review |

## Advanced Usage

### Review Against Specific Branch

Tell the agent what branch to compare against:
```
@uncommitted-review Compare against develop branch, not main
```

### Focus on Specific Files
```
@uncommitted-review Only review src/auth/*.py files
```

### Ignore Test Files
```
@uncommitted-review Ignore all test files, only review source code
```

### Code Quality Gate for CI

In GitHub Actions:
```yaml
- name: Review uncommitted changes
  run: |
    opencode run "@uncommitted-review" > review.md
    
- name: Check for critical issues
  run: |
    if grep -q "## Critical Issues" review.md; then
      echo "Critical issues found, failing build"
      cat review.md
      exit 1
    fi
```

### Custom Output Format

Modify the agent's output format section to integrate with your tools:

```markdown
## Output Format

Present your review in JSON format for CI integration:

\`\`\`json
{
  "score": 7,
  "ready": false,
  "issues": [
    {
      "severity": "critical",
      "file": "src/auth.py",
      "line": 42,
      "title": "SQL Injection",
      "description": "...",
      "suggestion": "..."
    }
  ]
}
\`\`\`
```

## Best Practices

### 1. Run Frequently
- Review after significant changes (not every small tweak)
- Review before pushing to remote
- Review before pull requests to catch issues early

### 2. Act on Feedback
- Fix Critical and High issues before committing
- Consider Medium issues (fix or document why not)
- Low issues are suggestions, not requirements

### 3. Use with Human Review
- This agent is a tool, not a replacement for code review
- Use it to catch obvious issues before human review
- Combine with peer review for best results

### 4. Customize for Your Team
- Adjust focus areas based on your priorities
- Add project-specific patterns to what to check
- Configure severity thresholds for your standards

### 5. Learn from Feedback
- Note recurring issues to improve patterns
- Update the agent's prompt with project-specific guidance
- Share findings with team to prevent similar issues

## Troubleshooting

### Agent not found
- Check file is in `~/.config/opencode/agent/` or `.opencode/agent/`
- Restart OpenCode after adding the agent
- Verify filename doesn't have special characters

### No changes detected
- Ensure you have uncommitted changes (`git status`)
- Check you're in a git repository
- Verify changes are tracked (not in `.gitignore`)

### Too much output
- Increase `num_max_findings` equivalent in the prompt
- Raise severity threshold to Medium or High
- Add file exclusions in your instructions

### False positives
- Add project-specific exceptions to the prompt
- Use `temperature: 0.0` for more deterministic results
- Document patterns the agent should ignore

## Contributing

Found improvements? Consider:
1. Adjusting the prompt for better results
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
