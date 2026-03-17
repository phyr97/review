# Finding-Constitution

Jedes Finding MUSS alle folgenden Prinzipien bestehen. Wenn ein Finding auch nur eines verletzt, wird es verworfen.

## Prinzip 1: Problem, nicht Beobachtung

Ein Finding beschreibt ein konkretes Problem. "Die Migration ist reversibel" oder "Es gibt gute Tests" sind Beobachtungen, keine Findings. Positive Aspekte gehoeren nicht in ein Review.

## Prinzip 2: Durch die Aenderung eingefuehrt

Das Problem muss durch den aktuellen Diff eingefuehrt oder verschlimmert worden sein. Pre-existing Issues die nicht vom Diff beruehrt wurden, sind kein Finding.

## Prinzip 3: Code-Verifikation

Das Finding muss durch tatsaechliches Lesen des Codes (Read-Tool) verifiziert sein. Ein Finding das ausschliesslich aus dem Diff abgeleitet wurde ohne den umgebenden Code zu pruefen, ist ungueltig. Der Diff zeigt Aenderungen, aber nicht den Kontext in dem sie leben.

## Prinzip 4: Intent-Respekt

Wenn die Aenderung den expliziten Zweck hat, Verhalten zu entfernen, umzubauen oder zu ersetzen, dann ist "Verhalten wurde entfernt/geaendert" kein Finding. Der Reviewer muss den Intent der Aenderung respektieren.

## Prinzip 5: Konkretes Fehlerszenario

Jedes Finding muss ein konkretes Szenario benennen in dem der Code fehlschlaegt. "Koennte problematisch sein" oder "ist nicht ideal" reichen nicht. Welcher Input, welcher Zustand, welcher Pfad fuehrt zum Fehler?

## Prinzip 6: Praezise Lokalisierung

Jedes Finding muss Datei:Zeile und den konkreten fehlerhaften Code-Pfad benennen. Vage Verweise auf "den neuen Code" oder "die Aenderung" sind unzulaessig.

## Prinzip 7: Kein Lob als Finding

Vorhandene Tests, saubere Migrationen, gute Architektur, konsistente Patterns sind keine Findings. Ein Review meldet Probleme oder schweigt. Stille ist ein valides Ergebnis.

## Zusammenfassung fuer Agent-Prompts

Bevor du ein Finding meldest, pruefe:
1. Ist es ein Problem oder eine Beobachtung?
2. Wurde es durch diesen Diff eingefuehrt?
3. Habe ich den tatsaechlichen Code gelesen (nicht nur den Diff)?
4. Widerspricht es dem erklaerten Zweck der Aenderung?
5. Kann ich ein konkretes Fehlerszenario benennen?
6. Kann ich Datei:Zeile und Code-Pfad angeben?
7. Ist es Kritik oder verstecktes Lob?

Wenn eine dieser Fragen das Finding disqualifiziert: verwerfen. Null Findings bei sauberem Code ist ein Erfolg.
