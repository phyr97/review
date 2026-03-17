# Chain-of-Verification (CoVe) Prozess

Self-Review-Schritt nach der Synthese. Jedes Finding das die Synthese ueberlebt hat, durchlaeuft diesen Prozess bevor es in den Output kommt.

## Ablauf

Fuer jedes Finding nach der Synthese (Phase 6):

### Schritt 1: Verifikationsfragen generieren

Formuliere 3 spezifische Fragen die das Finding bestaetigen oder widerlegen wuerden. Die Fragen muessen mit dem Read- oder Grep-Tool gegen den tatsaechlichen Code beantwortbar sein.

### Schritt 2: Fragen isoliert beantworten

Beantworte jede Frage einzeln durch Code-Lektuere. Nicht aus dem Gedaechtnis oder dem Diff, sondern durch tatsaechliches Lesen der relevanten Dateien.

### Schritt 3: Entscheidung

- Mindestens 2 von 3 Fragen bestaetigen das Problem: Finding behalten
- Weniger als 2 Bestaetigung: Finding verwerfen
- Frage nicht beantwortbar (Datei existiert nicht, Code nicht auffindbar): zaehlt als "nicht bestaetigt"

## Beispiel-Fragen nach Finding-Typ

### Bug-Finding
1. Gibt es an der gemeldeten Stelle tatsaechlich den beschriebenen Code-Pfad?
2. Wird der problematische Wert vorher validiert oder abgefangen?
3. Existiert ein Test der diesen Pfad abdeckt?

### Konsistenz-Finding
1. Verwenden die Nachbar-Module tatsaechlich das behauptete Pattern?
2. Gibt es eine dokumentierte Konvention (CLAUDE.md, Styleguide) die das Pattern vorschreibt?
3. Fuehrt die Abweichung zu einem konkreten Problem oder ist sie nur anders?

### Security-Finding
1. Erreicht User-Input tatsaechlich die gemeldete Stelle ohne Sanitization?
2. Existiert ein Sicherheitsmechanismus an anderer Stelle (Plug, Middleware, Validierung)?
3. Ist der Angriffspfad in der Produktionskonfiguration tatsaechlich erreichbar?

### Performance-Finding
1. Wird die betroffene Query/Operation tatsaechlich in einem Hot Path aufgerufen?
2. Ist die Datenmenge realistisch gross genug fuer das behauptete Problem?
3. Gibt es bereits Caching, Pagination oder andere Mitigation?

## Wichtig

- Der CoVe-Prozess ersetzt nicht die vorherigen Filter, er ergaenzt sie
- Findings die den CoVe-Prozess nicht bestehen werden stillschweigend verworfen, nicht herabgestuft
- Ziel ist Praezision: lieber ein echtes Finding als fuenf fragwuerdige
