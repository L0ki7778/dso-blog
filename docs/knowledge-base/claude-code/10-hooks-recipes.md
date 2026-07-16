# Hooks: Recipes

Fertige Konfigurationsbeispiele für gängige Hook-Anwendungsfälle. Grundlagen
(Events, Exit-Codes, Speicherort) stehen in [Hooks](./09-hooks.md); die
vollständige Event-Tabelle und Matcher-Syntax in
[Hooks: Erweitert](./11-hooks-erweitert.md).

## Benachrichtigung, wenn Claude eine Eingabe braucht

Desktop-Benachrichtigung, sobald Claude fertig ist und auf die nächste
Eingabe wartet – praktisch, um währenddessen an etwas anderem zu arbeiten,
ohne das Terminal im Blick behalten zu müssen. Nutzt das `Notification`-Event,
das auslöst, wenn Claude auf Eingabe oder eine Berechtigung wartet. In
`~/.claude/settings.json`:

```json title="~/.claude/settings.json (macOS)"
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

```json title="~/.claude/settings.json (Linux)"
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Claude Code needs your attention'"
          }
        ]
      }
    ]
  }
}
```

```json title="~/.claude/settings.json (Windows/PowerShell)"
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -Command \"[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('Claude Code needs your attention', 'Claude Code')\""
          }
        ]
      }
    ]
  }
}
```

Ein leerer `matcher` löst bei jeder Benachrichtigungsart aus. Um nur auf
bestimmte Ereignisse zu reagieren, einen dieser Werte setzen:

| Matcher | Löst aus, wenn … |
|---|---|
| `permission_prompt` | Claude eine Tool-Nutzung bestätigt haben will |
| `idle_prompt` | Claude fertig ist und auf den nächsten Prompt wartet |
| `auth_success` | die Authentifizierung abgeschlossen ist |
| `elicitation_dialog` | ein MCP-Server ein Eingabeformular öffnet |
| `elicitation_complete` | ein solches Formular abgeschickt oder verworfen wird |
| `elicitation_response` | die Antwort darauf zurück an den MCP-Server geht |
| `agent_needs_input` | eine Hintergrund-Session auf Eingabe wartet (nur bei geöffnetem Agent View) |
| `agent_completed` | eine Hintergrund-Session fertig ist oder fehlschlägt (nur bei geöffnetem Agent View) |

`agent_needs_input` und `agent_completed` erfordern Claude Code v2.1.198 oder
neuer.

## Code nach Edits automatisch formatieren

[Prettier](https://prettier.io/) automatisch auf jede von Claude bearbeitete
Datei anwenden, damit die Formatierung konsistent bleibt. Nutzt `PostToolUse`
mit einem `Edit|Write`-Matcher, läuft also nur nach Datei-Edits. `jq`
extrahiert den bearbeiteten Pfad, der an Prettier weitergereicht wird. In
`.claude/settings.json` im Projekt-Root:

```json title=".claude/settings.json"
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

Ab Claude Code v2.1.191 funktioniert im Matcher auch ein Komma statt `|`
(`"Edit,Write"`) – beide Trennzeichen sind für Tool-Namen-Matcher gleichwertig.

:::tip
Die Bash-Beispiele auf dieser Seite nutzen `jq` zum JSON-Parsen. Installation:
`brew install jq` (macOS), `apt-get install jq` (Debian/Ubuntu), sonst siehe
die [jq-Downloads](https://jqlang.github.io/jq/download/).
:::

## Edits an geschützten Dateien blockieren

Verhindert, dass Claude sensible Dateien wie `.env`, `package-lock.json` oder
alles unter `.git/` verändert. Claude bekommt eine Begründung zurück, warum
der Edit blockiert wurde, und kann seinen Ansatz anpassen.

**1. Hook-Skript anlegen** – `.claude/hooks/protect-files.sh`:

```bash title=".claude/hooks/protect-files.sh"
#!/bin/bash
# protect-files.sh

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

**2. Ausführbar machen** (macOS/Linux):

```bash
chmod +x .claude/hooks/protect-files.sh
```

**3. Hook registrieren** – ein `PreToolUse`-Hook in `.claude/settings.json`,
der vor jedem `Edit`- oder `Write`-Aufruf läuft:

```json title=".claude/settings.json"
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

## Kontext nach einer Kompaktierung neu einspeisen

Wenn Claudes Context-Fenster voll wird, fasst eine Kompaktierung den bisherigen
Verlauf zusammen – dabei können wichtige Details verloren gehen. Ein
`SessionStart`-Hook mit `compact`-Matcher speist nach jeder Kompaktierung
gezielt wieder Kontext ein.

Alles, was der Befehl auf stdout schreibt, wird Claudes Kontext hinzugefügt.
In `.claude/settings.json` im Projekt-Root:

```json title=".claude/settings.json"
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing. Current sprint: auth refactor.'"
          }
        ]
      }
    ]
  }
}
```

`echo` lässt sich durch jeden Befehl mit dynamischer Ausgabe ersetzen, z. B.
`git log --oneline -5` für die letzten Commits. Für Kontext bei **jedem**
Sessionstart eignet sich stattdessen eher [CLAUDE.md](./04-claude-md.md).

## Config-Änderungen protokollieren

Verfolgt, wenn sich Settings- oder Skill-Dateien während einer Session
ändern. Das `ConfigChange`-Event löst aus, wenn ein externer Prozess oder
Editor eine Konfigurationsdatei ändert – nützlich für Compliance-Logging oder
um unautorisierte Änderungen zu blockieren. In `~/.claude/settings.json`:

```json title="~/.claude/settings.json"
{
  "hooks": {
    "ConfigChange": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "jq -c '{timestamp: now | todate, source: .source, file: .file_path}' >> ~/claude-config-audit.log"
          }
        ]
      }
    ]
  }
}
```

Der Matcher filtert nach Konfigurationstyp: `user_settings`,
`project_settings`, `local_settings`, `policy_settings` oder `skills`. Um
eine Änderung zu blockieren, mit Exit-Code 2 beenden oder
`{"decision": "block"}` zurückgeben.

## Umgebung bei Verzeichnis- oder Dateiwechsel neu laden

Manche Projekte setzen je nach Verzeichnis unterschiedliche
Umgebungsvariablen. Tools wie [direnv](https://direnv.net/) übernehmen das
automatisch in der Shell – Claudes Bash-Tool bekommt solche Änderungen aber
nicht von selbst mit.

Die Kombination aus `SessionStart`- und `CwdChanged`-Hook behebt das:
`SessionStart` lädt die Variablen für das Startverzeichnis, `CwdChanged` lädt
sie bei jedem Verzeichniswechsel neu. Beide schreiben in `CLAUDE_ENV_FILE`,
das Claude Code vor jedem Bash-Befehl als Preamble ausführt. In
`~/.claude/settings.json`:

```json title="~/.claude/settings.json"
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "direnv export bash > \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ],
    "CwdChanged": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "direnv export bash > \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  }
}
```

`direnv allow` muss einmal in jedem Verzeichnis mit einer `.envrc`
ausgeführt werden, damit direnv sie laden darf. Bei devbox oder nix
funktioniert dasselbe Muster mit `devbox shellenv` bzw.
`devbox global shellenv` anstelle von `direnv export bash`.

Um stattdessen nur auf bestimmte Dateien statt jeden Verzeichniswechsel zu
reagieren, gibt es `FileChanged` mit einem `matcher`, der die zu
beobachtenden Dateinamen (getrennt durch `|`) auflistet. Dieses Beispiel
beobachtet `.envrc` und `.env` im Arbeitsverzeichnis:

```json title="~/.claude/settings.json"
{
  "hooks": {
    "FileChanged": [
      {
        "matcher": ".envrc|.env",
        "hooks": [
          {
            "type": "command",
            "command": "direnv export bash > \"$CLAUDE_ENV_FILE\""
          }
        ]
      }
    ]
  }
}
```

## Bestimmte Permission-Prompts automatisch bestätigen

Überspringt den Bestätigungsdialog für Tool-Calls, die immer erlaubt werden
sollen. Dieses Beispiel bestätigt automatisch `ExitPlanMode` – das Tool, das
Claude aufruft, wenn ein Plan fertig vorgestellt wurde und losgelegt werden
soll – damit nicht bei jedem fertigen Plan erneut nachgefragt wird.

Anders als bei den Exit-Code-Beispielen oben braucht die automatische
Bestätigung eine JSON-Entscheidung auf stdout. Ein `PermissionRequest`-Hook
löst aus, kurz bevor Claude Code einen Bestätigungsdialog anzeigen würde;
`"behavior": "allow"` beantwortet ihn stellvertretend.

Der Matcher grenzt den Hook auf `ExitPlanMode` ein, sodass keine anderen
Prompts betroffen sind. In `~/.claude/settings.json`:

```json title="~/.claude/settings.json"
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'"
          }
        ]
      }
    ]
  }
}
```

Bestätigt der Hook, verlässt Claude Code den Plan-Modus und stellt den vorher
aktiven Permission-Modus wieder her. Im Transcript erscheint an der Stelle
des Dialogs "Allowed by PermissionRequest hook". Dieser Weg behält immer die
laufende Konversation bei – anders als der Dialog kann er den Kontext nicht
leeren und eine frische Implementierungs-Session starten.

:::tip
Der Matcher sollte so eng wie möglich bleiben. `.*` oder ein leerer Matcher
würde **jeden** Permission-Prompt automatisch bestätigen – auch Datei-Writes
und Shell-Befehle.
:::

Um stattdessen einen bestimmten Permission-Modus zu setzen, kann die
Hook-Ausgabe ein `updatedPermissions`-Array mit einem `setMode`-Eintrag
enthalten. `mode` ist dabei ein beliebiger Permission-Modus (`default`,
`acceptEdits`, `bypassPermissions`, …), `destination: "session"` wendet ihn
nur für die aktuelle Session an:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow",
      "updatedPermissions": [
        { "type": "setMode", "mode": "acceptEdits", "destination": "session" }
      ]
    }
  }
}
```

:::tip
`bypassPermissions` greift nur, wenn dieser Modus für die Session bereits
verfügbar war (z. B. per `--dangerously-skip-permissions` gestartet oder
`permissions.defaultMode: "bypassPermissions"` in den Settings, sofern nicht
per `disableBypassPermissionsMode` deaktiviert) – er wird nie als
`defaultMode` gespeichert.
:::
