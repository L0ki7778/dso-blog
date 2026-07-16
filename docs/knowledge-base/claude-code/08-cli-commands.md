# CLI-Commands

Claude Code bringt eine Reihe von **Standard-Slash-Commands** mit, die in
jeder Installation verfügbar sind – unabhängig von Skills oder Plugins.
Skills (siehe [Skills](./07-skills.md)) können zusätzliche, projekt- oder
plugin-spezifische Befehle ergänzen (z. B. `/skill-creator`); diese Liste
umfasst nur die eingebauten Standardbefehle, sortiert nach Einsatzzweck.

## Einstellungen & Setup

| Befehl | Beschreibung |
|---|---|
| `/init` | Legt eine `CLAUDE.md` für das aktuelle Projekt an – durchsucht dabei das Repo automatisch (siehe [CLAUDE.md](./04-claude-md.md)). |
| `/permissions` | Öffnet die interaktive Verwaltung der Tool-Permissions (siehe [Permissions](./06-permissions.md)). |
| `/fewer-permission-prompts` | Wertet vergangene Sessions aus und schlägt neue `allow`-Regeln vor, damit seltener nach Bestätigung gefragt wird. |
| `/config` | Zeigt und ändert allgemeine CLI-Einstellungen. |
| `/add-dir` | Fügt der laufenden Session ein weiteres Arbeitsverzeichnis hinzu. |

## Model & Ressourcen

| Befehl | Beschreibung |
|---|---|
| `/model` | Wechselt das aktive Model, z. B. Opus, Sonnet oder Haiku (siehe [Models](./02-models.md)). |
| `/effort` | Wechselt das Effort-Level (siehe [Effort-Level](./03-effort.md)) – bricht dabei, wie ein Model-Wechsel, den Prompt-Cache. |
| `/fast` | Schaltet den Fast Mode um (Opus mit beschleunigter Ausgabe). |
| `/context` | Zeigt die aktuelle Context-Auslastung, aufgeschlüsselt nach Kategorie (System-Prompt, Tools, Skills, Messages, …). |
| `/compact` | Fasst den bisherigen Gesprächsverlauf zusammen, um wieder Context freizugeben. |
| `/clear` | Leert den Gesprächsverlauf vollständig und beginnt die Session neu. |

## Session & Verlauf

| Befehl | Beschreibung |
|---|---|
| `/cost` | Zeigt den bisherigen Token-Verbrauch der Session. |
| `/status` | Zeigt Account- und Systemstatus. |
| `/doctor` | Prüft die Claude-Code-Installation auf Probleme. |
| `/export` | Exportiert den aktuellen Gesprächsverlauf. |
| `/rewind` | Setzt Gespräch und/oder Codeänderungen auf einen früheren Punkt zurück. |
| `/todos` | Zeigt die aktuelle Aufgabenliste der Session. |
| `/usage` | Zeigt die verbleibenden Nutzungslimits des Plans. |

## Zusammenarbeit & Code

| Befehl | Beschreibung |
|---|---|
| `/agents` | Verwaltet spezialisierte Subagenten für bestimmte Aufgabentypen. |
| `/mcp` | Verwaltet Verbindungen zu MCP-Servern, inklusive OAuth-Login. |
| `/review` | Fordert einen Code-Review der aktuellen Änderungen an. |
| `/pr-comments` | Zeigt die Kommentare eines Pull Requests an. |
| `/bug` | Meldet einen Bug direkt an Anthropic. |

## Sonstiges

| Befehl | Beschreibung |
|---|---|
| `/help` | Zeigt eine Übersicht aller verfügbaren Befehle. |
| `/login` / `/logout` | Wechselt den verbundenen Anthropic-Account. |
| `/vim` | Aktiviert einen Vim-artigen Eingabemodus. |
| `/terminal-setup` | Richtet die Shift+Enter-Tastenkombination für Zeilenumbrüche im Terminal ein. |
| `/memory` | Öffnet `CLAUDE.md`/Memory-Dateien zur direkten Bearbeitung. |

:::tip
Bei Unsicherheit, was ein Befehl gerade tut oder welche Argumente er
akzeptiert, hilft `/help` immer als erster Anlaufpunkt.
:::
