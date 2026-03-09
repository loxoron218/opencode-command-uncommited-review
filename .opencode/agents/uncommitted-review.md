---
description: Reviews uncommitted changes for code quality issues
mode: subagent
hidden: true
temperature: 0.1
permission:
  write:
    "REVIEW*.md": allow
  edit: deny
  bash:
    "git diff*": allow
    "git log*": allow
    "git status": allow
    "git show": allow
  webfetch: deny
---