# Hooks

Hooks sind selbst definierte Shell-Befehle, die an bestimmten Punkten im
Lebenszyklus von Claude Code automatisch ausgeführt werden – z. B. nachdem
Claude eine Datei bearbeitet hat, wenn eine Aufgabe abgeschlossen ist, oder
wenn Claude auf Eingaben wartet.

Der Unterschied zu einem Skill (siehe [Skills](./07-skills.md)) ist die
**Verlässlichkeit**: Ein Skill ist ein zusätzlicher Prompt, den Claude
selbstständig anwenden *kann* – ein Hook läuft **immer**, unabhängig davon,
ob das Model sich gerade daran "erinnert". Damit eignen sich Hooks für
Dinge, die deterministisch passieren müssen: Formatierung erzwingen,
gefährliche Befehle blockieren, Benachrichtigungen verschicken, Projekt-Regeln
durchsetzen.

Für Entscheidungen, die eher Urteilsvermögen als eine feste Regel brauchen,
gibt es außerdem **Prompt-** und **Agent-basierte Hooks**, die dafür ein
Claude-Model heranziehen – siehe [Hooks: Erweitert](./11-hooks-erweitert.md).

## Den ersten Hook einrichten

Ein Hook wird über einen `hooks`-Block in einer Settings-Datei angelegt
(siehe [Speicherort und Reichweite](#speicherort-und-reichweite) weiter
unten). Das folgende Beispiel richtet eine Desktop-Benachrichtigung ein, die
auslöst, sobald Claude auf eine Eingabe wartet.

**1. Hook in die Settings eintragen** – `~/.claude/settings.json`,
`Notification`-Event, macOS-Beispiel mit `osascript`:

```json title="~/.claude/settings.json"
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

Existiert bereits ein `hooks`-Schlüssel in der Settings-Datei, wird
`Notification` als weiterer Event-Schlüssel daneben ergänzt, statt das
gesamte Objekt zu ersetzen:

```json title="~/.claude/settings.json"
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }]
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'" }]
      }
    ]
  }
}
```

Alternativ kann Claude selbst gebeten werden, den gewünschten Hook zu
schreiben – einfach in der CLI beschreiben, was er tun soll.

**2. Konfiguration prüfen** – `/hooks` öffnet den Hooks-Browser: eine Liste
aller verfügbaren Events mit der Anzahl konfigurierter Hooks je Event. Unter
`Notification` sollte der neue Eintrag auftauchen; ausgewählt zeigt er Event,
Matcher, Typ, Quelldatei und Befehl.

**3. Hook testen** – mit `Esc` zurück zur CLI, dann Claude etwas tun lassen,
das eine Berechtigung braucht, und währenddessen das Terminal wechseln. Es
sollte eine Desktop-Benachrichtigung erscheinen.

:::tip
Das `/hooks`-Menü ist **nur lesend**. Um Hooks hinzuzufügen, zu ändern oder
zu entfernen, muss die Settings-JSON direkt bearbeitet werden – oder Claude
wird gebeten, die Änderung vorzunehmen.
:::

## Speicherort und Reichweite

Wo ein Hook eingetragen wird, bestimmt seinen Geltungsbereich:

| Speicherort | Geltungsbereich | Teilbar? |
|---|---|---|
| `~/.claude/settings.json` | Alle eigenen Projekte | Nein, lokal auf der eigenen Maschine |
| `.claude/settings.json` | Einzelnes Projekt | Ja, kann mit dem Repo committet werden |
| `.claude/settings.local.json` | Einzelnes Projekt | Nein, wird von Claude Code gitignored angelegt |
| Managed Policy Settings | Organisationsweit | Ja, admin-verwaltet |
| Plugin-`hooks/hooks.json` | Solange das Plugin aktiv ist | Ja, im Plugin gebündelt |
| Skill- oder Subagent-Frontmatter | Solange der Skill/Subagent aktiv ist | Ja, direkt in der Komponenten-Datei definiert |

`/hooks` listet alle konfigurierten Hooks gruppiert nach Event auf. Über
`"disableAllHooks": true` in einer Settings-Datei lassen sich alle Hooks
deaktivieren – in Managed Settings gesetzte Hooks laufen davon unabhängig
trotzdem weiter, außer `disableAllHooks` ist auch dort gesetzt.

Wird eine Settings-Datei direkt bearbeitet, während Claude Code läuft, greift
der File-Watcher die Änderung normalerweise automatisch auf.

## Wie ein Hook mit Claude Code kommuniziert

Hooks kommunizieren über **stdin, stdout, stderr und Exit-Codes** –
mehr nicht. Wenn ein Event auslöst, bekommt der Hook-Befehl die
Event-Daten als JSON auf stdin. Er wertet das aus und teilt Claude Code über
den Exit-Code mit, wie es weitergehen soll.

**Beispiel-Input** (ein `PreToolUse`-Hook, bevor Claude einen Bash-Befehl
ausführt):

```json
{
  "session_id": "abc123",
  "cwd": "/Users/sarah/myproject",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

Jeder Event-Typ liefert zusätzlich eigene Felder – `UserPromptSubmit`-Hooks
bekommen z. B. den `prompt`-Text, `SessionStart`-Hooks eine `source` (`startup`,
`resume`, `clear` oder `compact`). Die vollständige Event-Liste inklusive
Matcher-Feldern steht in [Hooks: Erweitert](./11-hooks-erweitert.md).

**Exit-Codes** bestimmen, was als Nächstes passiert:

- **0** – kein Einwand, die Aktion läuft normal weiter. Bei `PreToolUse`
  heißt das *nicht* automatisch "erlaubt" – der normale
  [Permission-Ablauf](./06-permissions.md) greift trotzdem. Bei
  `UserPromptSubmit`, `UserPromptExpansion` und `SessionStart` wird alles,
  was der Hook auf stdout schreibt, Claudes Kontext hinzugefügt.
- **2** – die Aktion wird **blockiert**. Der Grund gehört auf stderr; Claude
  bekommt ihn als Feedback und kann seinen Ansatz anpassen. Manche Events
  lassen sich nicht blockieren (`SessionStart`, `Setup`, `Notification`,
  u. a.) – dort zeigt Exit-Code 2 stderr nur dem Nutzer an, die Ausführung
  läuft weiter.
- **jeder andere Code** – die Aktion läuft weiter, im Transcript erscheint
  aber ein `<hook name> hook error` mit der ersten Zeile von stderr; der
  komplette stderr-Inhalt landet im Debug-Log.

```bash title=".claude/hooks/check-drop-table.sh"
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q "drop table"; then
  echo "Blocked: dropping tables is not allowed" >&2  # stderr wird Claudes Feedback
  exit 2                                               # exit 2 = Aktion blockieren
fi

exit 0  # exit 0 = keine Entscheidung, normaler Permission-Ablauf greift
```

### Strukturierte JSON-Ausgabe

Exit-Codes erlauben nur "blockieren" oder "nichts sagen". Für feinere
Kontrolle: mit Exit-Code 0 beenden und stattdessen ein JSON-Objekt auf stdout
schreiben.

:::tip
Entweder Exit-Code 2 mit stderr-Nachricht **oder** Exit-Code 0 mit JSON –
nicht mischen. Claude Code ignoriert die JSON-Ausgabe, wenn gleichzeitig
Exit-Code 2 gesetzt ist.
:::

Beispiel: ein `PreToolUse`-Hook lehnt einen Tool-Call ab und erklärt Claude
warum:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep for better performance"
  }
}
```

`permissionDecision` kennt bei `PreToolUse` drei Werte: `"allow"`
(überspringt die interaktive Rückfrage – Deny-/Ask-Regeln greifen trotzdem
weiterhin), `"deny"` (bricht ab, `permissionDecisionReason` geht an Claude)
und `"ask"` (zeigt die normale Rückfrage). Andere Events nutzen andere
Entscheidungsfelder – z. B. `PostToolUse`/`Stop` ein `decision: "block"` auf
oberster Ebene, `PermissionRequest` dagegen
`hookSpecificOutput.decision.behavior`. Details dazu und zu den restlichen
Events stehen in [Hooks: Erweitert](./11-hooks-erweitert.md).

Für konkrete, direkt einsetzbare Beispiele siehe
[Hooks: Rezepte](./10-hooks-rezepte.md).
