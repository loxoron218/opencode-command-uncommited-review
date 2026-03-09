---
description: Read-only git analysis agent
mode: subagent
hidden: true
temperature: 0.1
permission:
  write: deny
  edit: deny
  bash:
    "git diff*": allow
    "git log*": allow
    "git status": allow
    "git show": allow
  webfetch: deny
---