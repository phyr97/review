# False-Positive Liste

Diese Patterns duerfen NICHT als Finding gemeldet werden. Wenn ein Agent-Finding auf eines dieser Muster passt, wird es in der Synthese verworfen.

## Stil und Formatierung

- Naming-Convention-Praeferenzen (camelCase vs. snake_case), es sei denn das Projekt ist inkonsistent
- Zeilenlaenge oder Wrapping-Stil
- Import/Require-Reihenfolge
- Trailing Commas oder Semicolons
- Anzahl Leerzeilen zwischen Funktionen
- Kommentar-Stil oder -Platzierung

## Bewusste Patterns

- TODO/FIXME/HACK-Kommentare (das ist das Tracking des Autors)
- Hardcodierte Werte in Test-Dateien (Test-Fixtures sollen konkret sein)
- Phoenix/Ecto-Boilerplate aus Generatoren (Schema-Felder, Changeset-Casts)
- Standard-Framework-Callbacks mit Default-Implementierungen
- Wrapper-Funktionen die "nur delegieren" (existieren fuer API-Contracts)
- Dead Code in Test-Helpern/Fixtures

## Haeufige Ueber-Reports

- "Fehlende Dokumentation" bei internen/privaten Funktionen
- "Koennte einen beschreibenderen Namen haben" bei Loop-Variablen (i, x, acc, etc.)
- "Koennte in eine Funktion extrahiert werden" bei 3-5 Zeilen Bloecken
- "Magic Number" bei offensichtlichen Werten (HTTP-Statuscodes, gaengige Limits wie 100, 1000)
- "Fehlendes Error Handling" bei Operationen die absichtlich crashen (let it crash)
- "Unused Variable" bei Variablen mit Underscore-Prefix
- "Pattern Matching verwenden" wenn if/case gleich klar ist
- "Fehlende Type Spec" in Projekten die Dialyzer nicht konsistent nutzen

## Framework-spezifisch

- Phoenix: Auto-generierte Migration-Timestamps, Default-Endpoint-Config, Standard-Router-Macros
- Ecto: Standard Changeset-Pipe (cast |> validate), Schema-Macro-Calls
- Ecto: UUID-Felder werden als Strings verglichen, das ist korrektes Verhalten (keine Binary-Vergleich-Warnung)
- LiveView: mount/3 returning {:ok, socket}, handle_event/3 Pattern Matching
- LiveView: LiveComponent-IDs sind pro Modul geschoped, keine Kollision zwischen verschiedenen Modulen
- ExUnit: `use ExUnit.Case, async: true` Boilerplate, describe/test Nesting

## Pre-existing Issues

- Probleme die bereits vor dem Diff existierten und nicht durch den Diff veraendert wurden
- Issues auf Zeilen die der Autor nicht modifiziert hat
- Bekannte Limitierungen die als Issues im Tracker stehen

## Linter/Compiler-Bereich

- Alles was ein Linter, Typechecker oder Compiler faengt (fehlende Imports, Type Errors, Formatierung)
- Es ist sicher anzunehmen dass CI diese Checks separat ausfuehrt

## Test-Code

- Vereinfachte Assertions in Tests
- Test-Setup das Produktionscode dupliziert (bewusste Isolation)
- Mock/Stub-Implementierungen die Validierung ueberspringen
- Hardcodierte Testdaten (Daten, IDs, Strings)

## Positive Beobachtungen

Folgendes sind keine Findings, auch wenn sie bemerkenswert sind:

- Vorhandene Tests fuer neue Features (das ist erwartet, kein Finding)
- Reversible Migrationen (das ist gute Praxis, kein Problem)
- Saubere Architektur oder gute Modul-Trennung (Lob gehoert nicht ins Review)
- Konsistente Nutzung bestehender Patterns (Konformitaet ist kein Finding)
- Umfassende Error-Handling-Abdeckung (das ist korrekt, nicht reportwuerdig)
