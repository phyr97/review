# Review Output Template

## Header

```
## Code Review: <branch-name oder "Staged Changes" oder "Commit <sha>">

Basis: <base-branch> | Diff: <N> Zeilen | Dateien: <N>
Agents: <N> (<Agent-Namen>) oder "Direkt-Review"
Stack: <erkannter Stack oder "Generisch">
```

## Severity-Zaehlzeile (immer wenn Findings vorhanden)

```
⚠ <N> Blocker · ⚡ <N> Warnungen · ℹ <N> Hinweise
```

Severity-Buckets ohne Findings einfach weglassen (`⚡ 0 Warnungen` nicht anzeigen).

## Findings (Zwei-Ebenen-Format)

Ziel: jeder Finding ist in einer Zeile entscheidbar, Details kommen on-demand.

### Blocker

Pro Blocker zwei Zeilen plus Aktions-Marker:

```
**<Nummer>. <kurzer Titel> in `<datei>:<zeile>`**
<1-Zeilen-Begruendung — was bricht konkret>
Aktion: [<N>F] Fix-Vorschlag · [<N>V] Vertiefen · [<N>I] Ignorieren
```

### Warnungen

Pro Warnung eine Zeile plus Aktions-Marker:

```
- [<N>] `<datei>:<zeile>` — <Kurztext> · [<N>F][<N>V][<N>I]
```

### Hinweise

Nicht direkt anzeigen. Nur Zaehler plus Prompt:

```
<N> Hinweise verfuegbar. Anzeigen? [J/N]
```

Wenn der User mit `J`, `ja`, "anzeigen", oder einer expliziten Nummer reagiert, die Hinweise im gleichen Einzeiler-Format wie Warnungen ausgeben.

## Aktions-Marker — Konvention

- `[<N>F]` Fix-Vorschlag — produziert einen Diff oder Code-Block im Chat. **Niemals Edit/Write/Bash-Modifikation.**
- `[<N>V]` Vertiefen — laenger erklaeren, Read-Only-Recherche im Code, mehr Kontext.
- `[<N>I]` Ignorieren — Finding aus dem Output streichen, kein weiteres Erwaehnen.

Beide Eingabeformen werden akzeptiert: `1V` ODER "vertiefe finding 1" ODER "Finding 1 vertiefen". Bei mehrdeutiger Eingabe nachfragen statt raten.

Im Plan/Text-Review (Phase 7) wird `[<N>F]` durch `[<N>Ä]` (Aenderungs-Vorschlag) ersetzt — sonst gleiches Schema.

## Beispiel-Output

```
## Code Review: feature/foo

Basis: develop | Diff: 245 Zeilen | Dateien: 7
Agents: 2 (bug-scanner, context-checker) | Stack: Elixir/Phoenix

⚠ 1 Blocker · ⚡ 2 Warnungen · ℹ 4 Hinweise

### Blocker

**1. Race Condition in `payment_processor.ex:42`**
Bei parallelen Requests kann der Saldo doppelt verbucht werden.
Aktion: [1F] Fix-Vorschlag · [1V] Vertiefen · [1I] Ignorieren

### Warnungen

- [2] `payment_processor.ex:88` — Input-Validation fehlt fuer `amount` · [2F][2V][2I]
- [3] `checkout_live.ex:120` — `assign_async` ohne Cancel-Handling · [3F][3V][3I]

### Hinweise
4 Hinweise verfuegbar. Anzeigen? [J/N]

---
Tipp: "1V" oder "vertiefe finding 1" — beides funktioniert.
```

## Keine-Findings-Output

Kein Header-Badge, kein Zaehler, keine Aktions-Marker:

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

Restrisiken sind keine Findings und tragen keine Aktions-Marker.

## Footer

```
---
Optionen:
1. MR-Beschreibung generieren
2. Einzelnes Finding vertiefen (oder Aktions-Marker nutzen)
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
