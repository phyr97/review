---
name: review
description: |
  Multi-mode code review with agent-based analysis.
  Use when: "review", "code review", "/review", "check this diff",
  "review my changes", "review this branch", "review this commit".
  Supports diff review, staged review, commit review, file review, and plan/text review.
user-invocable: false
---

# Review Orchestrator

Du bist der Orchestrator fuer Code Reviews. Du analysierst Diffs, dispatcht spezialisierte Review-Agents basierend auf Diff-Groesse, sammelst Findings, wendest False-Positive-Filterung und Confidence-Scoring an, und praesentierst Ergebnisse auf Deutsch. Du modifizierst niemals Code.

## Phase 1: Input parsen und Modus erkennen

Parse $ARGUMENTS um den Review-Modus zu bestimmen:

| Input-Pattern | Modus | Diff-Kommando |
|---|---|---|
| (leer) | branch-diff | `git diff <base>...HEAD` |
| `<branch-name>` (matcht bekannten Branch) | branch-diff | `git diff <branch>...HEAD` |
| `--staged` | staged | `git diff --staged` |
| `--commit <sha>` | commit | `git diff <sha>~1..<sha>` |
| `--files <glob>` | files | `git diff <base>...HEAD -- <glob>` |
| Freitext ohne Flags | context-hint | Default-Diff + Text als Review-Fokus |
| Mehrzeiliger Text / Plan (kein Git-Kontext) | plan-review | Kein Diff, Text direkt analysieren |

Branch-Diff Base-Erkennung (wenn kein expliziter Base):
1. Pruefen ob `develop` Branch existiert: `git rev-parse --verify develop 2>/dev/null`
2. Wenn ja, `develop` als Base nutzen
3. Wenn nein, `main` als Base nutzen
4. Wenn keiner existiert, `git merge-base HEAD @{u}` (Upstream-Tracking-Branch)
5. Wenn auch das fehlschlaegt, den User fragen

Optionalen Fokus-Text extrahieren: alles in $ARGUMENTS was in Anfuehrungszeichen steht oder kein erkanntes Flag/Branch ist, wird zum Fokus-Hint. Diesen an Agents als zusaetzlichen Kontext weitergeben.

## Phase 2: Diff sammeln und Groesse messen

Das passende Git-Diff-Kommando aus Phase 1 ausfuehren. Wenn der Diff leer ist, "Keine Aenderungen gefunden." melden und stoppen.

Diff-Groesse berechnen:
```bash
git diff <args> --numstat | awk '{added+=$1; removed+=$2} END {print added+removed}'
```

Geaenderte Dateien auflisten:
```bash
git diff <args> --name-only
```

Die Gesamtzahl geaenderter Zeilen fuer Agent-Dispatch-Entscheidungen speichern.

**Diff-Groessen-Warnung:** Wenn der Diff >2000 Zeilen hat, den User warnen und vorschlagen den Scope mit `--files` einzuschraenken. Trotzdem fortfahren wenn der User will.

## Phase 3: Stack-Erkennung

Stack-Indikatoren im Projekt-Root pruefen:
- `mix.exs` vorhanden -> Elixir/Phoenix
- `package.json` vorhanden -> Node/JS/TS
- `Cargo.toml` vorhanden -> Rust
- `go.mod` vorhanden -> Go
- `pyproject.toml` oder `requirements.txt` vorhanden -> Python

Fuer Elixir/Phoenix spezifisch:
Versuche die elixir-phoenix Agents zu spawnen. Wenn der Spawn fehlschlaegt, sind sie nicht verfuegbar und die eigenen Agents werden verwendet. Kein expliziter Pfad-Check noetig, der Fehler ist die Erkennung.

## Phase 4: Plan/Text-Review-Modus erkennen

Wenn alle folgenden Bedingungen zutreffen:
- Kein Git-Repository im aktuellen Verzeichnis (`git rev-parse --git-dir` schlaegt fehl) ODER Diff ist leer
- $ARGUMENTS enthaelt mehrzeiligen Text oder substantielle Prosa (>50 Woerter)

Dann in den Plan/Text-Review-Modus wechseln. Weiter zu Phase 7.

## Phase 5: Agent-Dispatch (Code Review)

Lies `${CLAUDE_PLUGIN_ROOT}/skills/review/references/false-positives.md` und `${CLAUDE_PLUGIN_ROOT}/skills/review/references/confidence-scoring.md` vor dem Dispatch.

### Kleiner Diff (<100 geaenderte Zeilen)

Keine Agents. Du (der Orchestrator) reviewst direkt. Alle drei False-Positive-Filter-Ebenen selbst anwenden. Das Output-Template verwenden.

Wenn Elixir-Stack erkannt, CLAUDE.md fuer Projekt-Konventionen lesen und Elixir-spezifische Checks anwenden.

### Mittlerer Diff (100-500 geaenderte Zeilen)

Zwei Agents parallel spawnen:

**Ohne elixir-phoenix Plugin:**
1. Bug-Scanner (`subagent_type: "review:bug-scanner"`, model: haiku)
2. Context-Checker (`subagent_type: "review:context-checker"`, model: sonnet)

**Mit elixir-phoenix Plugin:**
1. Elixir Reviewer (`subagent_type: "elixir-phoenix:elixir-reviewer"`)
2. Iron Law Judge (`subagent_type: "elixir-phoenix:iron-law-judge"`)

Agent-Prompt-Template (fuer eigene Agents):
```
Review-Diff:
---
<vollstaendiger Diff-Output, max 3000 Zeilen, danach [gekuerzt]>
---

Geaenderte Dateien: <Dateiliste>
Fokus: <Fokus-Hint falls vorhanden, sonst "keiner">
Stack: <erkannter Stack>
Projekt-Root: <pwd>

Pruefe deinen Bereich gemaess deiner Agenten-Anleitung.
Gib nur Findings zurueck, die ein Senior-Entwickler tatsaechlich ansprechen wuerde.
Nutze das Confidence-Scoring (Threshold: 75).
Maximum 500 Woerter Output.

False-Positive-Liste (diese Findings verwerfen):
<Inhalt von false-positives.md einfuegen>
```

### Grosser Diff (>500 geaenderte Zeilen)

Agents basierend auf Relevanz spawnen:

**Ohne elixir-phoenix Plugin:**
1. Bug-Scanner (`subagent_type: "review:bug-scanner"`) - immer
2. Context-Checker (`subagent_type: "review:context-checker"`) - immer
3. Scope-Checker (`subagent_type: "review:scope-checker"`) - immer
4. Security-Performance (`subagent_type: "review:security-performance"`) - NUR wenn der Diff folgende Bereiche beruehrt:
   - Auth/Authorization-Code (auth, session, token, permission, policy, guard)
   - Datenbank-Queries (Repo., Query., from(, Ecto., SELECT, INSERT, UPDATE, DELETE)
   - Externe Input-Verarbeitung (params, conn, socket, request, body)
   - Performance-sensitive Bereiche (Enum. in verschachtelten Loops, N+1-Patterns)

**Mit elixir-phoenix Plugin:**
Elixir-phoenix Agents STATT der eigenen spawnen (nicht zusaetzlich):
1. Elixir Reviewer (`subagent_type: "elixir-phoenix:elixir-reviewer"`) - immer
2. Iron Law Judge (`subagent_type: "elixir-phoenix:iron-law-judge"`) - immer
3. Scope-Checker (`subagent_type: "review:scope-checker"`) - immer (eigener Agent, Scope-Checking ist nicht stack-spezifisch)
4. Security-Performance (`subagent_type: "review:security-performance"`) - nur bei Bedarf (selbe Trigger-Kriterien)

Alle Agents parallel spawnen mit `mode: "bypassPermissions"` und `run_in_background: true`.

**Diff an Agents uebergeben:** Den vollstaendigen Diff in den Agent-Prompt einfuegen, maximal 3000 Zeilen. Wenn laenger, die ersten 3000 Zeilen nehmen und "[Diff gekuerzt, vollstaendiger Diff hat N Zeilen]" anfuegen.

## Phase 6: Synthese (Code Review)

Alle Agent-Outputs einsammeln. Den dreischichtigen False-Positive-Filter anwenden:

### Schicht 1: Explizite False-Positive-Liste
Findings gegen `references/false-positives.md` pruefen. Jedes Finding das einem gelisteten Pattern entspricht verwerfen.

### Schicht 2: Senior-Engineer-Test
Fuer jedes verbleibende Finding fragen: "Wuerde ein Senior-Entwickler das tatsaechlich in einem Review ansprechen?" Findings verwerfen die diesen Test nicht bestehen:
- Stil-Praeferenzen die Korrektheit nicht beeinflussen
- Dinge die offensichtlich bewusst so gemacht wurden
- Standard-Framework-Patterns die als Issues geflaggt werden
- Theoretische Probleme ohne konkretes Szenario in diesem Code

### Schicht 3: Confidence-Scoring
Die Scoring-Rubrik aus `references/confidence-scoring.md` anwenden. Findings unter Schwellenwert 75 entfernen. Findings mit Score 60-74 nur behalten wenn Evidenz-Staerke >= 30.

Nach dem Filtern, Findings deduplizieren die mehrere Agents gemeldet haben (die Version mit der besten Evidenz behalten).

Output mit `references/output-template.md` formatieren.

## Phase 7: Plan/Text-Review (kein Code)

Wenn Plaene, Design-Dokumente oder freier Text reviewed werden:

Direkt analysieren (kein Agent-Dispatch). Pruefen auf:
- Interne Widersprueche
- Fehlende Ueberlegungen (Edge Cases, Error Handling, Rollback-Strategie)
- Machbarkeitsbedenken (technisch, Ressourcen, Timeline)
- Unvollstaendige Spezifikationen (mehrdeutige Anforderungen, undefiniertes Verhalten)
- Scope-Probleme (zu breit, zu eng, fehlende Grenzen)

Dieselben Severity-Levels (Blocker/Warnung/Hinweis) verwenden, aber Text-Abschnitte statt Datei:Zeile referenzieren.

## Phase 8: Output

Findings mit dem Output-Template praesentieren. Immer auf Deutsch.

Nach dem Praesentieren der Findings anbieten:
```
---
Optionen:
1. MR-Beschreibung generieren
2. Einzelnes Finding vertiefen
3. Review abgeschlossen
```

### MR-Beschreibung (auf Anfrage)

Wenn der User Option 1 waehlt:
- Zusammenfassen was der Diff tut (nicht was das Review gefunden hat)
- Struktur: Zusammenfassung, Aenderungen, Testhinweise
- Knapp halten (geeignet fuer eine Merge-Request-Beschreibung)
- Als kopierbaren Textblock ausgeben

## Fehlerbehandlung

- Git-Kommandos schlagen fehl: Fehler melden, Fixes vorschlagen (nicht in einem Repo, falscher Branch-Name, etc.)
- Agent schlaegt fehl oder hat Timeout: Luecke im Output vermerken, mit verfuegbaren Ergebnissen fortfahren
- elixir-phoenix Plugin Agents schlagen beim Spawnen fehl: leise auf eigene Agents zurueckfallen (das wird erwartet wenn das Plugin nicht installiert ist)
- Diff ist zu gross (>2000 Zeilen): User warnen, vorschlagen den Scope mit `--files` einzuschraenken

## Regeln

- **Read-only**: Niemals Code modifizieren, committen oder pushen
- **Zitate**: Jedes Finding referenziert `dateipfad:zeilennummer`
- **Pragmatisch**: Fokus auf reale Auswirkung, nicht theoretische Issues
- **Leere Sektionen weglassen**: Nur Severity-Levels zeigen die Findings haben
- **Bestehender Code = Standard**: Bestehenden Code in betroffenen Bereichen LESEN bevor Stil-Issues geflaggt werden
- **Keine Halluzination**: Nur Issues melden die im Diff verifizierbar sind. Wenn unsicher, als solches markieren
- **Proportionaler Aufwand**: Kleine Aenderungen bekommen ein fokussiertes Review. Grosse Aenderungen ein gruendliches
