---
description: Review code diff, staged changes, commit, or plan/text
argument-hint: [base-branch] [--staged] [--commit sha] [--files glob] ["focus"]
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(wc:*), Bash(test:*), Bash(awk:*), Agent
model: sonnet
---

Starte einen Review mit dem review Skill.

Kontext:
- Argumente: $ARGUMENTS
- Aktueller Branch: !`git branch --show-current 2>/dev/null || echo "kein git repo"`
- Verfuegbare Branches: !`git branch --list 2>/dev/null | head -20 || echo "keine"`
- Working Tree Status: !`git status --short 2>/dev/null | head -30 || echo "kein git repo"`

Fuehre den Review gemaess der Skill-Anleitung durch. Lies zuerst den Skill und die Referenzdateien, dann folge den Phasen.
