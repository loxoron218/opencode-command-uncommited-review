# Quick Reference: Updated Agent System

## What's New

The uncommitted review agents now work together with **automatic delegation**:

```
User runs: @uncommitted-review
    ↓
Main agent analyzes changes
    ↓
Intelligently delegates to subagents:
    • @security-review (for auth, input, queries, etc.)
    • @performance-review (for loops, databases, I/O, etc.)
    ↓
Integrates all findings
    ↓
Presents comprehensive review
```

## Files Updated

1. **opencode-agent-uncommitted-review.md** - Main orchestrator agent
2. **opencode-agent-security-review.md** - Security specialist (hidden)
3. **opencode-agent-performance-review.md** - Performance specialist (hidden)

## Usage

### Standard Review (Recommended)
```
@uncommitted-review
```

**This will:**
- Automatically invoke security subagent if needed
- Automatically invoke performance subagent if needed
- Integrate all findings into cohesive review
- Show which subagents were invoked

### Direct Subagent Invocation (Optional)
```
@security-review Focus on authentication code
@performance-review Check database queries
```

**Note**: Subagents are hidden from autocomplete menu but still accessible via direct invocation.

## What Changed vs Before

| Aspect | Before | After |
|---------|--------|-------|
| Invocation | User had to run 3 separate commands | Single command triggers delegation |
| Coordination | No, manual effort | Automatic, LLM decides |
| Output | Fragmented across 3 reviews | Integrated into single review |
| Subagents visible | All in autocomplete | Hidden from autocomplete |

## Restart Required

```bash
# Exit OpenCode and restart to load changes
opencode
```

## Example Output

```markdown
## Summary
Modified 4 files adding JWT authentication and user session management.

## Critical Issues
[Issues from main agent and subagents]

## High Priority Issues
[Issues from main agent and subagents]

## Overall Assessment
- Quality Score: 7/10
- Security Score: 8/10 (from @security-review)
- Performance Score: 6/10 (from @performance-review)
- Ready to commit: No
- Required actions: Fix JWT secret hardcoding and optimize session lookup

---
**Note**: This review was enhanced by specialized subagents:
- @security-review: invoked
- @performance-review: invoked
```

## Documentation

- **AGENT_UPDATE_SUMMARY.md** - Detailed technical documentation
- **agent-flow.md** - Visual flow diagrams
- **README.md** - Full project documentation
- **QUICKSTART.md** - Getting started guide

## Troubleshooting

### Subagents not invoking
1. Verify files are in `~/.config/opencode/agent/`
2. Restart OpenCode completely
3. Try with security/performance-sensitive code to trigger delegation

### Missing scores in output
Scores only appear when subagents are invoked. If changes are simple/minimal, delegation is skipped and scores won't show.

## Need Help?

- Check `AGENT_UPDATE_SUMMARY.md` for technical details
- See `agent-flow.md` for visual flow
- Review `README.md` for full documentation
