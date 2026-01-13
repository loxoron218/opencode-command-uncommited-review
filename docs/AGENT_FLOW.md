# Agent Delegation Flow

```
User Request
    |
    v
@uncommitted-review
    |
    v
┌─────────────────────────────────┐
│  Main Agent Review Process    │
└─────────────────────────────────┘
    |
    v
Check git diff --stat
    |
    v
┌─────────────────────────────────┐
│  Evaluate Changes              │
│                               │
│  Security-sensitive? ───────► Yes ──► @security-review
│                               │               |
│                               │               v
│                               │         Return Security Findings
│                               │               |
│                               ◄───────────────┘
│                               │
│  Performance-sensitive? ───► Yes ──► @performance-review
│                               │               |
│                               │               v
│                               │         Return Performance Findings
│                               │               |
│                               ◄───────────────┘
│                               │
│  No delegation needed ──► General Analysis
│                               │
└─────────────────────────────────┘
    |
    v
┌─────────────────────────────────┐
│  Integrate All Findings       │
│  - Main agent analysis          │
│  - Security subagent results   │
│  - Performance subagent results  │
└─────────────────────────────────┘
    |
    v
┌─────────────────────────────────┐
│  Generate Final Review         │
│  - Summary                    │
│  - All issues by severity      │
│  - Security score              │
│  - Performance score           │
│  - Ready to commit decision   │
└─────────────────────────────────┘
    |
    v
Present to User
```

## Decision Tree

```
Start: Analyze git diff
    │
    ├─► Has user input? ──► @security-review
    ├─► Has auth/session? ──► @security-review
    ├─► Has database queries? ──► @security-review
    ├─► Has crypto/secrets? ──► @security-review
    │
    ├─► Has nested loops? ──► @performance-review
    ├─► Has large data ops? ──► @performance-review
    ├─► Has file I/O? ──► @performance-review
    ├─► Has API calls? ──► @performance-review
    │
    ├─► Minimal changes? ──► General analysis only
    ├─► Config/docs only? ──► General analysis only
    │
    └─► General analysis (always performed)
```

## Output Structure

```
## Summary
[What changed + what it does]

## Critical Issues
[From main agent + subagents]

## High Priority Issues
[From main agent + subagents]

## Medium Priority Issues
[From main agent + subagents]

## Low Priority Issues
[From main agent only]

## Suggestions
[From all sources]

## Overall Assessment
- Quality Score: X/10 [main agent]
- Security Score: X/10 [@security-review]
- Performance Score: X/10 [@performance-review]
- Ready to commit: Yes/No [main agent decision]
- Required actions: [...]

---
**Note**: This review was enhanced by specialized subagents:
- @security-review: [invoked/not invoked]
- @performance-review: [invoked/not invoked]