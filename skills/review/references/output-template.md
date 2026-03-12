# Review Output Template

## Header

```
## Code Review: <branch-name oder "Staged Changes" oder "Commit <sha>">

Basis: <base-branch> | Diff: <N> Zeilen | Dateien: <N>
Agents: <N> (<Agent-Namen>) oder "Direkt-Review"
Stack: <erkannter Stack oder "Generisch">
```

## Findings (nach Severity sortiert)

```
### Blocker

#### <kurzer Titel>
- Datei: `<pfad>:<zeile>`
- Was: <was ist falsch>
- Warum: <warum ist das ein Problem>
- Fix-Richtung: <wie beheben>

### Warnungen

#### <kurzer Titel>
...

### Hinweise

#### <kurzer Titel>
...
```

Leere Sektionen weglassen. Nur Severity-Levels mit Findings anzeigen.

## Keine-Findings-Output

```
Keine Review-Findings.

Geprueft: <N> Dateien, <N> Zeilen
Agents: <liste>
```

## Restrisiken (immer wenn zutreffend)

```
### Restrisiken
- <nicht abgedeckte Bereiche, z.B. "Integration mit externem API nicht verifizierbar ohne Laufzeitkontext">
- <Test-Luecken, z.B. "Keine Tests fuer den neuen Edge Case vorhanden">
```

## Footer

```
---
Optionen:
1. MR-Beschreibung generieren
2. Einzelnes Finding vertiefen
3. Review abgeschlossen
```

## MR-Beschreibung (auf Anfrage)

```
## Zusammenfassung
<2-3 Saetze was dieser MR tut>

## Aenderungen
- <Aenderung 1>
- <Aenderung 2>

## Testhinweise
- <was manuell testen>
- <was automatisierte Tests abdecken>
```
