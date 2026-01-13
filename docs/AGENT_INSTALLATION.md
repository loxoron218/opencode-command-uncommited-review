# Agent Installation Notes

## Issue Fixed

Original agent files had YAML syntax issues that prevented OpenCode from starting:

### Problematic Pattern:
```yaml
tools:
  write: false
  edit: false
permission:
  task:
    "*": deny
```

### Fixed Pattern:
```yaml
permission:
  write: deny
  edit: deny
  bash:
    "*": deny
    "git diff": allow
```

## Installed Agents

Three agents are now installed in `~/.config/opencode/agent/`:

1. **uncommitted-review.md** - General code quality review
2. **security-review.md** - Security vulnerability review
3. **performance-review.md** - Performance optimization review

## Testing

Restart OpenCode to load these agents:

```bash
# Kill existing opencode if running
pkill opencode

# Start fresh
opencode
```

Once OpenCode starts, you should see these agents available:
- Type `@` and press Tab to see autocomplete
- Agents: `uncommitted-review`, `security-review`, `performance-review`

## Usage

Test them in OpenCode:

```
@uncommitted-review
```

Or:
```
@security-review
```

Or:
```
@performance-review
```

## File Permissions

Files should be readable (644):
```bash
chmod 644 ~/.config/opencode/agent/*.md
```

## Troubleshooting

If OpenCode still doesn't start:

1. **Move agents temporarily**:
   ```bash
   mkdir -p ~/backup-agents
   mv ~/.config/opencode/agent/*.md ~/backup-agents/
   ```

2. **Start OpenCode** - should work now
3. **Add back one at a time**:
   ```bash
   cp ~/backup-agents/uncommitted-review.md ~/.config/opencode/agent/
   opencode  # Test
   ```

4. **Check logs** if issue persists:
   ```bash
   opencode --debug  # Or check ~/.config/opencode/logs/
   ```

## Configuration Reference

All agents use this structure:
- **mode**: `subagent` (invoked via @mention)
- **temperature**: Low (0.1) for consistent, focused output
- **model**: Anthropic Claude Sonnet 4
- **permission**: Read-only (write/edit denied), bash restricted to git commands only

This ensures safe, read-only code review without accidentally modifying files.
