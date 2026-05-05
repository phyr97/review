# Chain-of-Verification (CoVe) Prozess

Self-Review-Schritt nach der Synthese. Jedes Finding das die Synthese ueberlebt hat, durchlaeuft diesen Prozess bevor es in den Output kommt.

## Ablauf

Fuer jedes Finding nach der Synthese (Phase 6):

### Schritt 1: Verifikationsfragen generieren

Formuliere 4 Fragen pro Finding:

1. **Drei Code-Verifikationsfragen** — sie pruefen die *Existenz* des Problems im Code. Mit Read- oder Grep-Tool gegen die tatsaechlichen Dateien beantwortbar. Beispiele weiter unten.
2. **Eine Logik-Frage** — sie pruefe die *logische Zwingbarkeit* der Schlussfolgerung, nicht die Code-Existenz. Template:

> *"Ist die Schlussfolgerung des Findings logisch zwingend aus dem Code, oder nur plausibel? Welche Alternativ-Erklaerung (Steelman) wuerde das Finding entkraeften?"*

Die Logik-Frage ist nicht durch Code-Lektuere beantwortbar — sie ist eine Steelman-Pruefung gegen das Finding selbst.

### Schritt 2: Fragen isoliert beantworten

- Code-Verifikationsfragen: einzeln durch Code-Lektuere beantworten. Nicht aus dem Gedaechtnis oder dem Diff, sondern durch tatsaechliches Lesen der relevanten Dateien.
- Logik-Frage: explizit den staerksten Steelman gegen das Finding formulieren. Wenn dieser Steelman plausibel und konsistent mit dem Code ist, "bricht" er das Finding.

### Schritt 3: Entscheidung

Zwei unabhaengige Checks:

**Code-Verifikation (wie bisher):**
- Mindestens 2 von 3 Fragen bestaetigen das Problem: bestanden
- Weniger als 2 Bestaetigung: nicht bestanden, Finding **verwerfen** (stillschweigend)
- Frage nicht beantwortbar (Datei existiert nicht, Code nicht auffindbar): zaehlt als "nicht bestaetigt"

**Logik-Frage (neu):**
- Steelman entkraeftet das Finding NICHT (Schlussfolgerung ist logisch zwingend): bestanden, Finding bleibt mit voller Severity
- Steelman entkraeftet das Finding (alternative Erklaerung ist plausibel): **Severity-Downgrade um eine Stufe**
  - Blocker → Warnung
  - Warnung → Hinweis
  - Hinweis → verwerfen

Die beiden Checks gelten kumulativ:
- Code-Verifikation nicht bestanden → verwerfen, egal was die Logik-Frage sagt
- Code-Verifikation bestanden + Logik-Frage bestanden → Finding bleibt
- Code-Verifikation bestanden + Logik-Frage entkraeftet → Severity-Downgrade

## Beispiel-Fragen nach Finding-Typ

### Bug-Finding

Code-Verifikation:
1. Gibt es an der gemeldeten Stelle tatsaechlich den beschriebenen Code-Pfad?
2. Wird der problematische Wert vorher validiert oder abgefangen?
3. Existiert ein Test der diesen Pfad abdeckt?

Logik-Frage:
- "Koennte der Author das absichtlich so gemacht haben (Defensiv-Programmierung, dokumentierte Annahme, bewusster Trade-off)?"

### Konsistenz-Finding

Code-Verifikation:
1. Verwenden die Nachbar-Module tatsaechlich das behauptete Pattern?
2. Gibt es eine dokumentierte Konvention (CLAUDE.md, Styleguide) die das Pattern vorschreibt?
3. Fuehrt die Abweichung zu einem konkreten Problem oder ist sie nur anders?

Logik-Frage:
- "Ist die Abweichung absichtlich, weil der Kontext es erfordert (z.B. Performance, Lesbarkeit, externer Zwang)?"

### Security-Finding

Code-Verifikation:
1. Erreicht User-Input tatsaechlich die gemeldete Stelle ohne Sanitization?
2. Existiert ein Sicherheitsmechanismus an anderer Stelle (Plug, Middleware, Validierung)?
3. Ist der Angriffspfad in der Produktionskonfiguration tatsaechlich erreichbar?

Logik-Frage:
- "Ist der Angriffspfad in der erwarteten Production-Konfiguration tatsaechlich erreichbar (Auth, Network-Boundary, Feature-Flag)?"

### Performance-Finding

Code-Verifikation:
1. Wird die betroffene Query/Operation tatsaechlich in einem Hot Path aufgerufen?
2. Ist die Datenmenge realistisch gross genug fuer das behauptete Problem?
3. Gibt es bereits Caching, Pagination oder andere Mitigation?

Logik-Frage:
- "Ist die Datenmenge fuer dieses Modul realistisch gross genug, dass das Problem in Production sichtbar wird?"

## Wichtig

- Der CoVe-Prozess ersetzt nicht die vorherigen Filter, er ergaenzt sie
- Findings die die Code-Verifikation nicht bestehen werden stillschweigend verworfen
- Findings die die Logik-Frage nicht bestehen werden herabgestuft, nicht verworfen (ausser sie waren schon Hinweis)
- Ziel ist Praezision: lieber ein echtes Finding als fuenf fragwuerdige
- Severity-Downgrades sind im Output sichtbar (Finding bleibt, nur in der niedrigeren Severity-Sektion)
