# Hooks: Erweitert

Vollständige Event-Übersicht, Matcher-Syntax, die drei alternativen
Hook-Typen (Prompt, Agent, HTTP) sowie Grenzen und Troubleshooting.
Grundlagen und ein erstes Beispiel stehen in [Hooks](./09-hooks.md), fertige
Konfigurationen in [Hooks: Rezepte](./10-hooks-rezepte.md).

## Alle Hook-Events

Ein Event feuert an einem bestimmten Punkt im Lebenszyklus. Wenn es auslöst,
laufen alle passenden Hooks parallel; identische Hook-Befehle werden dabei
automatisch dedupliziert.

| Event | Löst aus, wenn … |
|---|---|
| `SessionStart` | eine Session beginnt oder fortgesetzt wird |
| `Setup` | Claude Code mit `--init-only` gestartet wird, oder mit `--init`/`--maintenance` im `-p`-Modus – für einmalige Vorbereitung in CI/Skripten |
| `UserPromptSubmit` | ein Prompt abgeschickt wird, bevor Claude ihn verarbeitet |
| `UserPromptExpansion` | ein getippter Befehl zu einem Prompt expandiert, bevor er Claude erreicht – kann die Expansion blockieren |
| `PreToolUse` | bevor ein Tool-Call ausgeführt wird – kann ihn blockieren |
| `PermissionRequest` | ein Permission-Dialog erscheinen würde |
| `PermissionDenied` | ein Tool-Call vom Auto-Mode-Classifier abgelehnt wurde. `{retry: true}` erlaubt Claude einen erneuten Versuch |
| `PostToolUse` | nachdem ein Tool-Call erfolgreich war |
| `PostToolUseFailure` | nachdem ein Tool-Call fehlgeschlagen ist |
| `PostToolBatch` | nachdem ein komplettes Batch paralleler Tool-Calls abgeschlossen ist, vor dem nächsten Model-Aufruf |
| `Notification` | Claude Code eine Benachrichtigung sendet |
| `MessageDisplay` | während Assistant-Text angezeigt wird |
| `SubagentStart` | ein Subagent gestartet wird |
| `SubagentStop` | ein Subagent fertig ist |
| `TaskCreated` | ein Task über `TaskCreate` angelegt wird |
| `TaskCompleted` | ein Task als abgeschlossen markiert wird |
| `Stop` | Claude mit der Antwort fertig ist |
| `StopFailure` | der Turn wegen eines API-Fehlers endet – Output und Exit-Code werden ignoriert |
| `TeammateIdle` | ein Agent-Team-Mitglied gleich in den Leerlauf geht |
| `InstructionsLoaded` | eine `CLAUDE.md` oder `.claude/rules/*.md` geladen wird – bei Sessionstart und bei verzögert nachgeladenen Dateien |
| `ConfigChange` | sich eine Konfigurationsdatei während der Session ändert |
| `CwdChanged` | sich das Arbeitsverzeichnis ändert, z. B. durch einen `cd`-Befehl |
| `FileChanged` | eine beobachtete Datei sich auf der Platte ändert – der `matcher` legt fest, welche Dateinamen |
| `WorktreeCreate` | ein Worktree über `--worktree` oder `isolation: "worktree"` angelegt wird – ersetzt das Standard-Git-Verhalten |
| `WorktreeRemove` | ein Worktree entfernt wird – bei Sessionende oder wenn ein Subagent fertig ist |
| `PreCompact` | vor einer Context-Kompaktierung |
| `PostCompact` | nach einer abgeschlossenen Kompaktierung |
| `Elicitation` | ein MCP-Server während eines Tool-Calls eine Nutzereingabe anfordert |
| `ElicitationResult` | nachdem der Nutzer auf eine solche Anfrage geantwortet hat, bevor die Antwort zurück an den Server geht |
| `SessionEnd` | eine Session beendet wird |

Jeder Hook hat einen `type`, der bestimmt, wie er läuft. Die meisten nutzen
`"type": "command"` (Shell-Befehl). Vier weitere Typen:

- `"type": "http"` – Event-Daten per POST an eine URL, siehe [HTTP-Hooks](#http-hooks).
- `"type": "mcp_tool"` – ruft ein Tool auf einem bereits verbundenen MCP-Server auf.
- `"type": "prompt"` – Ein-Turn-LLM-Auswertung, siehe [Prompt-basierte Hooks](#prompt-basierte-hooks).
- `"type": "agent"` – Mehr-Turn-Verifikation mit Tool-Zugriff (experimentell), siehe [Agent-basierte Hooks](#agent-basierte-hooks).

### Mehrere Hooks am selben Event

Bei mehreren Hooks auf demselben Event läuft jeder Hook-Befehl vollständig
durch, bevor Claude Code die Ergebnisse zusammenführt. Ein `deny` eines Hooks
stoppt die anderen Hooks nicht – Nebenwirkungen eines anderen Hooks lassen
sich also nicht darauf verlassen, dass ein `deny` sie unterdrückt.

Nach Abschluss aller passenden Hooks werden die Ergebnisse kombiniert: bei
`PreToolUse`-Permission-Entscheidungen gilt die restriktivste Antwort, in der
Reihenfolge `deny`, `defer`, `ask`, `allow`. Text aus `additionalContext`
wird von jedem Hook übernommen und gemeinsam an Claude weitergegeben.

Beispiel: zwei `PreToolUse`-Hooks auf `Bash` – der erste protokolliert jeden
Befehl und beendet mit Exit-Code 0, der zweite blockiert mit Exit-Code 2,
sobald der Befehl `rm -rf` enthält:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "jq -r .tool_input.command >> ~/.claude/bash.log" },
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm-rf.sh" }
        ]
      }
    ]
  }
}
```

Ruft Claude `rm -rf /tmp/build` auf, laufen beide Hooks parallel: der
Logging-Hook schreibt den Befehl ins Log und meldet Exit-Code 0 (keine
Entscheidung), der Guard-Hook blockiert mit Exit-Code 2. Das `deny` hat
Vorrang, der Log-Eintrag bleibt aber trotzdem bestehen, weil dieser Hook
schon gelaufen war.

## Matcher-Syntax

Ohne Matcher feuert ein Hook bei jedem Vorkommen seines Events. Matcher
grenzen das ein – z. B. soll ein Formatter nur nach Datei-Edits laufen,
nicht nach jedem Tool-Call:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "prettier --write ..." }]
      }
    ]
  }
}
```

Jeder Event-Typ filtert dabei auf ein anderes Feld:

| Event | Worauf der Matcher filtert | Beispielwerte |
|---|---|---|
| `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied` | Tool-Name | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | wie die Session gestartet wurde | `startup`, `resume`, `clear`, `compact` |
| `Setup` | welches CLI-Flag Setup ausgelöst hat | `init`, `maintenance` |
| `SessionEnd` | warum die Session endete | `clear`, `resume`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other` |
| `Notification` | Benachrichtigungstyp | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response`, `agent_needs_input`, `agent_completed` |
| `SubagentStart`, `SubagentStop` | Agent-Typ | `general-purpose`, `Explore`, `Plan` oder eigene Agent-Namen |
| `PreCompact`, `PostCompact` | was die Kompaktierung ausgelöst hat | `manual`, `auto` |
| `ConfigChange` | Konfigurationsquelle | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills` |
| `StopFailure` | Fehlertyp | `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, `unknown` |
| `InstructionsLoaded` | Ladegrund | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact` |
| `Elicitation`, `ElicitationResult` | MCP-Servername | konfigurierte MCP-Servernamen |
| `FileChanged` | zu beobachtende Dateinamen (Literale, kein Regex) | `.envrc\|.env` |
| `UserPromptExpansion` | Befehlsname | eigene Skill- oder Befehlsnamen |
| `UserPromptSubmit`, `PostToolBatch`, `Stop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `WorktreeCreate`, `WorktreeRemove`, `CwdChanged`, `MessageDisplay` | kein Matcher-Support | feuert immer bei jedem Vorkommen |

Das `"Edit|Write"`-Matcher-Beispiel oben feuert nur bei `Edit`- oder
`Write`-Tool-Calls, nicht bei `Bash`, `Read` o. ä. Ab Claude Code v2.1.191
ist ein Komma (`"Edit,Write"`) gleichbedeutend mit `|` als Trennzeichen für
Tool-Namen-Matcher.

Claude kann Dateien auch über die `Bash`-Tool per Shell-Befehl anlegen oder
ändern. Wenn ein Hook wirklich **jede** Dateiänderung sehen muss (z. B. für
Compliance-Scans), empfiehlt sich zusätzlich ein `Stop`-Hook, der den
Arbeitsbaum einmal pro Turn scannt – bzw. zusätzlich `Bash` matchen und mit
`git status --porcelain` geänderte/untrackte Dateien auflisten.

MCP-Tools folgen einer eigenen Namenskonvention:
`mcp__<server>__<tool>` (z. B. `mcp__github__search_repositories`). Ein
Regex-Matcher wie `mcp__github__.*` trifft alle Tools eines Servers,
`mcp__.*__write.*` serverübergreifend alle "write"-artigen Tools.

### Nach Tool-Name und Argumenten filtern: das `if`-Feld

Das `if`-Feld nutzt dieselbe Syntax wie [Permission-Regeln](./06-permissions.md)
und filtert Tool-Name **und** Argumente zusammen, sodass der Hook-Prozess nur
bei einer echten Übereinstimmung überhaupt gestartet wird. Das geht über
`matcher` hinaus, der nur auf Gruppenebene nach Tool-Name filtert.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(git *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-git-policy.sh"
          }
        ]
      }
    ]
  }
}
```

| `if`-Pattern | Bash-Befehl | Hook läuft? | Warum |
|---|---|---|---|
| `Bash(git *)` | `git push` | ja | Befehlsname passt |
| `Bash(git *)` | `npm test && git push` | ja | jeder Teilbefehl wird geprüft; `git push` passt |
| `Bash(git *)` | `echo $(git log)` | ja | Befehle in `$()`/Backticks werden geprüft; `git log` passt |
| `Bash(git *)` | `echo $(date)` | nein | kein Teilbefehl passt zu `git *` |
| `Bash(git push *)` | `echo $(date)` | ja | Pattern, die mehr als nur den Befehlsnamen angeben, lassen den Hook bei `$()`/Backticks/`$VAR` trotzdem laufen |

Lässt sich der Bash-Befehl nicht parsen, läuft der Hook sicherheitshalber
trotzdem (fail-open) – `if` ist also eine Best-Effort-Filterung. Für eine
harte Erlauben/Verbieten-Regel gehört das ins [Permission-System](./06-permissions.md),
nicht in einen Hook.

`if` funktioniert nur bei den Tool-Events: `PreToolUse`, `PostToolUse`,
`PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`. Bei jedem
anderen Event verhindert `if` sogar, dass der Hook überhaupt läuft.

## Prompt-basierte Hooks

Für Entscheidungen, die Urteilsvermögen statt einer festen Regel brauchen:
`type: "prompt"`. Statt einen Shell-Befehl auszuführen, schickt Claude Code
den eigenen Prompt zusammen mit den Hook-Eingabedaten an ein Claude-Model
(Default: Haiku) und lässt es entscheiden. Ein anderes Model lässt sich über
das `model`-Feld angeben.

Das Model liefert nur eine Ja/Nein-Entscheidung als JSON:

- `"ok": true` – die Aktion läuft weiter.
- `"ok": false` – abhängig vom Event: bei `Stop`/`SubagentStop` wird `reason`
  an Claude zurückgegeben, das weiterarbeitet; bei `PreToolUse` wird der
  Tool-Call abgelehnt und `reason` als Tool-Fehler an Claude gemeldet; bei
  `PostToolUse`, `PostToolBatch`, `UserPromptSubmit` und
  `UserPromptExpansion` endet der Turn, `reason` erscheint als Warnzeile im
  Chat.

Beispiel: ein `Stop`-Hook fragt das Model, ob wirklich alle Aufgaben
abgeschlossen sind. Antwortet es mit `"ok": false`, arbeitet Claude weiter
und nutzt `reason` als nächste Anweisung:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if all tasks are complete. If not, respond with {\"ok\": false, \"reason\": \"what remains to be done\"}."
          }
        ]
      }
    ]
  }
}
```

## Agent-basierte Hooks

:::tip
Agent-Hooks sind **experimentell** – Verhalten und Konfiguration können sich
noch ändern. Für produktive Workflows sind Command-Hooks vorzuziehen.
:::

Wenn die Verifikation Dateien lesen oder Befehle ausführen muss:
`type: "agent"`. Anders als Prompt-Hooks (ein einzelner LLM-Aufruf) startet
ein Agent-Hook einen Subagenten, der Dateien lesen, Code durchsuchen und
weitere Tools nutzen kann, bevor er eine Entscheidung zurückgibt.

Agent-Hooks nutzen dasselbe `"ok"`/`"reason"`-Antwortformat wie
Prompt-Hooks, aber mit einem längeren Standard-Timeout von 60 Sekunden und
bis zu 50 Tool-Use-Turns.

Beispiel: prüft vor dem Beenden, ob alle Unit-Tests durchlaufen:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results. $ARGUMENTS",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

Prompt-Hooks reichen, wenn die Eingabedaten des Hooks allein für die
Entscheidung genügen. Agent-Hooks lohnen sich, wenn etwas gegen den
tatsächlichen Zustand der Codebasis geprüft werden muss.

## HTTP-Hooks

`type: "http"` schickt die Event-Daten per POST an einen HTTP-Endpunkt,
statt einen Shell-Befehl auszuführen. Der Endpunkt bekommt dasselbe JSON,
das ein Command-Hook auf stdin bekäme, und liefert das Ergebnis im selben
JSON-Format über den Response-Body zurück.

Nützlich, wenn ein Webserver, eine Cloud-Function oder ein externer Dienst
die Hook-Logik übernehmen soll – z. B. ein gemeinsamer Audit-Service, der
Tool-Use-Events teamweit protokolliert:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/tool-use",
            "headers": { "Authorization": "Bearer $MY_TOKEN" },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  }
}
```

Der Endpunkt sollte im Response-Body dasselbe JSON-Format wie ein
Command-Hook zurückgeben. Um einen Tool-Call zu blockieren, muss eine
2xx-Antwort mit den passenden `hookSpecificOutput`-Feldern zurückkommen –
der HTTP-Statuscode allein blockiert nichts.

Header-Werte unterstützen `$VAR_NAME`- bzw. `${VAR_NAME}`-Interpolation.
Nur in `allowedEnvVars` gelistete Variablen werden aufgelöst, alle anderen
`$VAR`-Referenzen bleiben leer.

## Grenzen

- Command-Hooks kommunizieren ausschließlich über stdout, stderr und
  Exit-Codes – sie können keine `/`-Befehle oder Tool-Calls auslösen. Über
  `additionalContext` zurückgegebener Text wird als reiner Text in Claudes
  Kontext eingefügt. HTTP-Hooks kommunizieren stattdessen über den
  Response-Body.
- Timeouts variieren nach Typ (per `timeout`-Feld in Sekunden überschreibbar):
  `command`/`http`/`mcp_tool` 10 Minuten (`UserPromptSubmit` 30 Sekunden,
  `MessageDisplay` 10 Sekunden), `prompt` 30 Sekunden, `agent` 60 Sekunden.
- `PostToolUse`-Hooks können Aktionen nicht rückgängig machen, da das Tool
  schon gelaufen ist.
- `PermissionRequest`-Hooks feuern **nicht** im Non-Interactive-Modus
  (`-p`-Flag) – dort stattdessen `PreToolUse` für automatisierte
  Permission-Entscheidungen nutzen.
- `Stop`-Hooks feuern immer, wenn Claude mit der Antwort fertig ist, nicht
  nur bei "echtem" Task-Abschluss. Sie feuern nicht bei Nutzer-Interrupts;
  API-Fehler lösen stattdessen `StopFailure` aus.
- Geben mehrere `PreToolUse`-Hooks `updatedInput` zurück, um Tool-Argumente
  umzuschreiben, gewinnt der zuletzt fertige Hook – da Hooks parallel laufen,
  ist die Reihenfolge nicht deterministisch. Nicht mehr als einen Hook
  dieselben Tool-Eingaben verändern lassen.

### Hooks und Permission-Modi

`PreToolUse`-Hooks feuern **vor** jeder Prüfung des Permission-Modus. Ein
Hook, der `permissionDecision: "deny"` zurückgibt, blockiert das Tool auch
im `bypassPermissions`-Modus oder mit `--dangerously-skip-permissions` – so
lässt sich eine Policy durchsetzen, die Nutzer nicht durch einen
Modus-Wechsel umgehen können.

Umgekehrt gilt das nicht: ein Hook, der `"allow"` zurückgibt, hebelt keine
Deny-Regeln aus den Settings aus, und er kann auch keine Rückfrage bei
Connector-Tools unterdrücken, die eine Organisation auf `ask` gesetzt hat,
oder bei MCP-Tools mit `requiresUserInteraction`. Hooks können also
Beschränkungen verschärfen, aber nicht über das hinaus lockern, was die
Permission-Regeln erlauben.

## Troubleshooting

**Hook feuert nicht:**

- `/hooks` öffnen und prüfen, ob der Hook unter dem richtigen Event auftaucht.
- Matcher-Pattern prüfen – Matcher sind case-sensitive.
- Den richtigen Event-Typ verwenden: `PreToolUse` feuert **vor**
  Tool-Ausführung, `PostToolUse` danach.
- `PermissionRequest`-Hooks im Non-Interactive-Modus (`-p`) durch
  `PreToolUse` ersetzen.

**"`<hook name>` hook error" im Transcript:**

- Das Skript ist mit einem unerwarteten Exit-Code beendet – manuell testen:
  ```bash
  echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./my-hook.sh
  echo $?
  ```
- Bei "command not found": absolute Pfade oder `$CLAUDE_PROJECT_DIR`
  verwenden. Um Shell-Quoting ganz zu vermeiden, `"args": []` ergänzen –
  das Skript wird dann direkt ohne Shell gestartet.
- Bei "jq: command not found": `jq` installieren oder Python/Node.js zum
  JSON-Parsen nutzen.
- Skript ausführbar machen: `chmod +x ./my-hook.sh`.

**`/hooks` zeigt keine Hooks an, obwohl eine Settings-Datei bearbeitet
wurde:**

- Datei-Edits werden normalerweise automatisch übernommen; falls nach ein
  paar Sekunden nichts erscheint, hat der File-Watcher die Änderung
  eventuell verpasst – Session neu starten, um ein Reload zu erzwingen.
- JSON-Gültigkeit prüfen: trailing Commas und Kommentare sind nicht erlaubt.
- Speicherort prüfen: `.claude/settings.json` für Projekt-Hooks,
  `~/.claude/settings.json` für globale Hooks.

**Stop-Hook erreicht die Block-Grenze** (Claude arbeitet immer weiter, statt
zu stoppen, und der Turn endet mit einer Warnung, dass der Stop-Hook zu oft
in Folge blockiert hat):

Claude Code überschreibt einen Stop-Hook, nachdem er achtmal in Folge ohne
Fortschritt blockiert hat. Das Hook-Skript muss daher selbst prüfen, ob es
bereits eine Fortsetzung ausgelöst hat – das `stop_hook_active`-Feld aus dem
JSON-Input auswerten und bei `true` früh beenden:

```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Claude darf stoppen
fi
# … restliche Hook-Logik
```

**JSON-Validierung schlägt fehl, obwohl das Hook-Skript gültiges JSON
ausgibt:**

Bei einem Shell-Form-Command-Hook (ohne `args`) startet Claude Code
standardmäßig `sh -c` (macOS/Linux) bzw. Git Bash (Windows). Diese Shell ist
non-interactive, aber Git Bash und manche Konfigurationen (z. B. `BASH_ENV`
auf `~/.bashrc`) laden das Profil trotzdem. Enthält das Profil
bedingungslose `echo`-Aufrufe, landen die vor dem eigentlichen JSON:

```text
Shell ready on arm64
{"decision": "block", "reason": "Not allowed"}
```

Claude Code versucht das als JSON zu parsen und scheitert. Fix: `echo` im
Shell-Profil auf interaktive Shells beschränken:

```bash
# in ~/.zshrc oder ~/.bashrc
if [[ $- == *i* ]]; then
  echo "Shell ready"
fi
```

`$-` enthält die Shell-Flags, `i` steht für interaktiv – Hooks laufen
non-interactive, der `echo` wird also übersprungen.

**Debuggen:** Die Transcript-Ansicht (`Ctrl+O`) zeigt pro gefeuertem Hook
eine Zeile: Erfolg bleibt still, blockierende Fehler zeigen stderr,
nicht-blockierende Fehler einen `<hook name> hook error` mit der ersten
stderr-Zeile. Für die vollen Details (welche Hooks gematcht haben,
Exit-Codes, stdout, stderr) hilft das Debug-Log: mit
`claude --debug-file /tmp/claude.log` starten und in einem zweiten Terminal
`tail -f /tmp/claude.log` – oder mitten in der Session `/debug` ausführen,
um Logging zu aktivieren und den Log-Pfad anzuzeigen.
