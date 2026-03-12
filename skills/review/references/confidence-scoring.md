# Confidence-Scoring Rubrik

Jedes Finding bekommt einen Confidence-Score von 0 bis 100. Der Schwellenwert fuer die Aufnahme in den Output ist 75.

## Scoring-Faktoren

### Evidenz-Staerke (0-40 Punkte)

- 40: Reproduzierbarer, konkreter Code-Pfad fuehrt zum Problem
- 30: Starker Pattern-Match mit dokumentiertem Known-Bad-Pattern
- 20: Verdaechtiges Pattern, benoetigt Runtime-Kontext zur Bestaetigung
- 10: Theoretische Bedenken basierend auf allgemeinen Prinzipien
- 0: Reine Spekulation ohne Evidenz im Code

### Impact-Schwere (0-30 Punkte)

- 30: Datenverlust, Security-Breach, System-Crash in Produktion
- 20: Bug der Nutzer in gaengigen Workflows betrifft
- 15: Bug der Edge Cases oder seltene Workflows betrifft
- 10: Wartbarkeitsproblem mit konkreten zukuenftigen Kosten
- 5: Kleineres Qualitaetsproblem
- 0: Kosmetisch oder praeferenzbasiert

### Kontext-Passung (0-20 Punkte)

- 20: Finding ist spezifisch fuer DIESE Codebase und ihre Patterns
- 15: Finding gilt fuer diesen Stack/Framework generell
- 10: Finding ist eine generische Best Practice
- 5: Finding koennte auf jeden Code zutreffen
- 0: Finding ignoriert lokale Konventionen oder Kontext

### Einzigartigkeit (0-10 Punkte)

- 10: Neuartiges Finding das nicht von Lintern/CI/Standard-Tools abgedeckt wird
- 5: Teilweise von Tools abgedeckt aber fuegt Nuancen hinzu
- 0: Standard-Linter-Regel die CI fangen sollte

## Schwellenwert-Anwendung

- Score >= 75: In Output aufnehmen
- Score 60-74: Nur aufnehmen wenn Evidenz-Staerke >= 30
- Score < 60: Verwerfen

## Beispiele

"Fehlender nil-Check vor Map.get auf User-Input der nil sein koennte"
-> Evidenz: 35 (konkreter Code-Pfad) + Impact: 20 (User-facing Bug) + Kontext: 15 (Phoenix params) + Einzigartigkeit: 5 = 75 -> Aufnehmen

"Variable koennte einen beschreibenderen Namen haben"
-> Evidenz: 10 (subjektiv) + Impact: 5 (kosmetisch) + Kontext: 5 (generisch) + Einzigartigkeit: 0 = 20 -> Verwerfen

"SQL Injection via String-Interpolation in Ecto Fragment"
-> Evidenz: 40 (konkret) + Impact: 30 (Security) + Kontext: 20 (Ecto-spezifisch) + Einzigartigkeit: 10 = 100 -> Aufnehmen
