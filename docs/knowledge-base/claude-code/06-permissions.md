# Permissions

Claude Code prüft vor bestimmten Tool-Aufrufen, ob sie erlaubt sind. Diese
Permissions werden in `.claude/settings.json` definiert.

## Workspace-Permissions

Beispiel für eine `.claude/settings.json` in einem Projekt:

```json title=".claude/settings.json"
{
  "permissions": {
    // Array of permission rules to allow tool use
    "allow": ["Bash(npm run *)", "Bash(git commit *)"],

    // Array of permission rules to deny tool use
    "deny": ["Bash(git push *)"],

    // Array of permission rules to ask confirmation before tool use
    "ask": ["Bash(rm *)", "Bash(mv *)"],

    // "default", "plan", "auto", "dontAsk", "bypassPermissions"
    "defaultMode": "acceptEdits"
  }
}
```

- Permissions lassen sich auch interaktiv über `/permissions` in der Claude
  CLI erstellen.
- Der `/fewer-permission-prompts`-Befehl prüft die Transkripte vergangener
  Sessions, fasst häufig bestätigte Tool-Calls zusammen und schlägt passende
  Permission-Regeln vor, die unter `allow` aufgenommen werden könnten.
- Permission-Prompts lassen sich auch über einen `PreToolUse`- oder
  `PermissionRequest`-**Hook** automatisch beantworten (siehe
  [Hooks](./09-hooks.md) und [Hooks: Rezepte](./10-hooks-rezepte.md)) – ein
  solcher Hook kann Regeln durchsetzen, die selbst im
  `bypassPermissions`-Modus greifen.

## Globale (User-Level) Permissions

Im User-Ordner gibt es ebenfalls einen `.claude`-Ordner mit Permissions, die
lokal – also nicht nur für den aktuellen Workspace – gelten. Diese
User-Level-Permissions gelten **vor** den Workspace-Permissions:

```json title="~/.claude/settings.json"
{
  "permissions": {
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ],
    "disableBypassPermissionsMode": "disable"
  },
  "allowManagedPermissionRulesOnly": true
}
```
