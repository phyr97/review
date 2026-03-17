# review Plugin

Multi-mode Code-Review-Plugin fuer Claude Code. Dispatcht spezialisierte Review-Agents, filtert False Positives und wendet Confidence-Scoring an.

## Veroeffentlichung

Repo: `phyr97/review` (privat, GitHub)
Marketplace: `phyr97/phyr97-marketplace` (liegt unter `../phyr97-marketplace/`)

### Update veroeffentlichen

1. Version in `.claude-plugin/plugin.json` bumpen
2. Commit und Push:
```bash
git add -A && git commit -m "..." && git push origin main
```
3. Marketplace updaten (version in `marketplace.json` und `README.md` anpassen):
```bash
cd ../phyr97-marketplace && git add . && git commit -m "Bump review to X.Y.Z" && git push origin main
```
4. Plugin lokal updaten:
```bash
claude plugin update review
```

## Befehle

- `/review` - Branch-Diff reviewen
- `/review --staged` - Staged Changes reviewen
- `/review --commit <sha>` - Einzelnen Commit reviewen
- `/review --files <glob>` - Bestimmte Dateien reviewen
- `/review "Fokus-Text"` - Review mit Fokus-Hint

## Architektur

Orchestrator (SKILL.md) -> Agents (bug-scanner, context-checker, scope-checker, security-performance)
Synthese mit Intent-Filter, False-Positive-Listen, Confidence-Scoring und CoVe Self-Review.
