# Effort-Level

Neben dem Model selbst lässt sich auch das **Effort-Level** einstellen – es
bestimmt, wie viele Ressourcen das Model in eine Aufgabe steckt. Ein
niedriges Effort-Level lässt das Model mit weniger Aufwand (und günstiger)
arbeiten.

- Effort wird manuell über `/effort` gewechselt.
- Ein Effort-Wechsel mitten in der Session bricht – genau wie ein
  Model-Wechsel – den Prompt-Cache: Die CLI warnt vor dem Wechsel, dass der
  aktuelle Context dabei verloren geht. Effort sollte daher möglichst zu
  Sessionbeginn festgelegt werden, nicht mittendrin gewechselt.
- Antwortet ein Model nicht wie gewünscht, sollte zuerst das **Effort-Level
  erhöht** werden, bevor man zu einem stärkeren Model wechselt.

## `/advisor`

Das `/advisor`-Tool erlaubt punktuellen Zugriff auf ein stärkeres Model
(Opus), ohne das eigentliche Session-Model zu wechseln – man arbeitet
weiter mit dem aktuellen Model und ruft Opus nur über den Tool-Call auf.

## Context

- Der Context (Gesprächsverlauf, Dateien, Tool-Ergebnisse) liegt als Prefix
  im Prompt-Cache des Models.
- Die Standardgröße des Context liegt bei 200k Tokens und kann bis auf 1M
  Tokens erhöht werden.
- Der Context sollte trotzdem nicht zu groß werden – die Antwortqualität
  verschlechtert sich mit wachsendem Context.
- Die aktuelle Context-Größe lässt sich mit `/context` prüfen, mit
  `/compact` lässt sich der Context zusammenfassen.
