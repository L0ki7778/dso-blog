# Models

Claude besteht aus austauschbaren Modellen, die über die Anthropic API
zugänglich sind. Welches Modell sinnvoll ist, hängt von der Aufgabe ab:

| Model    | Wann verwenden                                                                                                             |
|----------|------------------------------------------------------------------------------------------------------------------------------|
| `Opus`   | Für Probleme mit versteckten Ursachen oder schwierige Design-Entscheidungen – wenn die Antwort tiefes Reasoning braucht und nicht einfach "im Code steht". |
| `Sonnet` | Für alltägliche Engineering-Aufgaben, bei denen die Lösung Codeverständnis erfordert.                                        |
| `Haiku`  | Für mechanische Arbeit – wenn keine Entscheidungen getroffen werden müssen, sondern nur festgelegte Schritte auszuführen sind. |

## Model wechseln

- Das Model wird in der CLI mit `/model` gewechselt.
- Modelle sind **stateless** – ein Wechsel des Models während einer
  laufenden Session bricht den Prompt-Cache. Am besten wird das Model daher
  zu Sessionbeginn festgelegt, nicht mittendrin gewechselt.
