# Skills

Skills erweitern Claude Code um wiederverwendbare Fähigkeiten – vergleichbar
mit einem zusätzlichen, fest hinterlegten Prompt, der bei Bedarf automatisch
mitgeliefert wird.

## Skills erstellen

- Neue Skills werden mit dem `/skill-creator`-Befehl in der Claude CLI
  erstellt.
- Alle Frontmatter-Felder sind technisch optional – ohne `name` wird der
  Ordnername verwendet. In der Praxis sollte `description` aber immer gesetzt
  werden, da Claude darüber entscheidet, ob und wann der Skill automatisch
  greift (siehe unten). Die vollständige Übersicht aller Properties steht
  unten.
- Skills liegen unter `.claude/skills/<skill-name>/SKILL.md`. Der
  Ordnername (`<skill-name>`) sollte mit dem `name` des Skills
  übereinstimmen.
- Ob und wann ein Skill selbstständig vom Harness verwendet wird, hängt
  maßgeblich von einer guten **Description** ab: Passt die aktuelle
  Situation zur Description, greift Claude auf den Skill zurück.
- Nach dem Erstellen eines neuen Skills muss der Client neu geladen werden
  (`/reload`).

## Config-Properties (Frontmatter)

Alle Einstellungen eines Skills stehen im YAML-Frontmatter am Anfang der
`SKILL.md`, zwischen den beiden `---`-Zeilen. Alle Felder sind optional;
lediglich `description` wird empfohlen, damit Claude überhaupt weiß, wann der
Skill zum Einsatz kommen soll.

| Property | Pflicht? | Beschreibung |
|---|---|---|
| `name` | Nein | Anzeigename in der Skill-Liste. Default: Ordnername. Weicht ggf. vom Befehl ab, mit dem der Skill aufgerufen wird. |
| `description` | Empfohlen | Was der Skill tut und wann er verwendet werden soll – entscheidet, ob Claude ihn automatisch zieht. Ohne Angabe wird der erste Markdown-Absatz verwendet. `description` + `when_to_use` werden in der Skill-Liste zusammen auf 1.536 Zeichen gekürzt – den wichtigsten Anwendungsfall daher zuerst nennen. |
| `when_to_use` | Nein | Zusätzlicher Kontext, wann Claude den Skill aufrufen soll, z. B. Trigger-Phrasen oder Beispielanfragen. Wird an `description` angehängt und zählt zum selben 1.536-Zeichen-Limit. |
| `argument-hint` | Nein | Hinweis in der Autovervollständigung auf erwartete Argumente, z. B. `[issue-number]` oder `[filename] [format]`. |
| `arguments` | Nein | Benannte, positionsbezogene Argumente für `$name`-Platzhalter im Skill-Inhalt (siehe [Argumente](#argumente)). Akzeptiert einen leerzeichengetrennten String oder eine YAML-Liste; die Namen werden der Reihe nach den Argument-Positionen zugeordnet. |
| `disable-model-invocation` | Nein | `true` verhindert, dass Claude den Skill automatisch lädt – Aufruf dann nur manuell über `/name`. Verhindert außerdem das Vorladen in Subagenten sowie (ab v2.1.196) die Ausführung, wenn ein geplanter Task den Skill als Prompt auslöst. Default: `false`. |
| `user-invocable` | Nein | `false` blendet den Skill aus dem `/`-Menü aus – gedacht für Hintergrundwissen, das Nutzer nicht direkt per Befehl aufrufen sollen. Default: `true`. |
| `allowed-tools` | Nein | Tools, die der Skill ohne Rückfrage nutzen darf, solange er aktiv ist, z. B. erlaubt `Bash(gh *)` nur `gh`-Aufrufe über Bash. String (leerzeichen-/kommagetrennt) oder YAML-Liste. |
| `disallowed-tools` | Nein | Tools, die entfernt werden, solange der Skill aktiv ist – z. B. `AskUserQuestion` bei einem autonomen Hintergrund-Loop, der nicht nachfragen soll. String oder YAML-Liste. Die Einschränkung gilt nur bis zur nächsten eigenen Nachricht. |
| `model` | Nein | Model, das verwendet wird, solange der Skill aktiv ist. Die Umstellung gilt nur für den **Rest des aktuellen Turns** und wird nicht in den Settings gespeichert – im nächsten Prompt gilt wieder das Session-Model. Akzeptiert dieselben Werte wie `/model`, zusätzlich `inherit` (behält das aktive Model bei). Ein durch die Organisations-Allowlist (`availableModels`) ausgeschlossener Wert wird ignoriert; die Session bleibt dann beim aktuellen Model. |
| `effort` | Nein | Effort-Level, solange der Skill aktiv ist – überschreibt das Session-Effort-Level (siehe [Effort-Level](./03-effort.md)). Default: erbt von der Session. Optionen: `low`, `medium`, `high`, `xhigh`, `max` – welche Stufen verfügbar sind, hängt vom Model ab. |
| `context` | Nein | `fork` lässt den Skill in einem eigenen, abgetrennten Subagent-Kontext laufen statt direkt im Hauptgespräch. |
| `agent` | Nein | Legt fest, welcher Subagent-Typ den Skill ausführt, wenn `context: fork` gesetzt ist, z. B. `Explore`. |
| `hooks` | Nein | Hooks, die an den Lebenszyklus dieses Skills gebunden sind (siehe [Hooks](./09-hooks.md) für das allgemeine Konfigurationsformat). |
| `paths` | Nein | Glob-Pattern, die einschränken, bei welchen Dateien der Skill automatisch aktiviert wird. String (kommagetrennt) oder YAML-Liste – nur bei einer Übereinstimmung lädt Claude den Skill automatisch. |
| `shell` | Nein | Shell für Inline-Befehle (`` !`command` `` und `` ```! ``-Blöcke, siehe unten) in diesem Skill. `bash` (Default) oder `powershell`. `powershell` führt Inline-Shell-Befehle unter Windows über PowerShell aus und erfordert `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`. |

## Argumente

Skills können Argumente entgegennehmen. Im Skill werden sie über
`$ARGUMENTS` referenziert:

```yaml title=".claude/skills/deploy/SKILL.md"
---
name: deploy
description: Deploys our codebase to either staging or production
---

# Deploy

1. Run the tests
2. Bundle the app
3. Deploy to $ARGUMENTS
```

Aufgerufen wird das z. B. mit `/deploy staging`.

Für mehrere, benannte Argumente gibt es statt eines einzigen `$ARGUMENTS`-Blobs
auch die Property `arguments`: Sie ordnet Namen der Reihe nach den
Argument-Positionen zu, sodass im Skill-Inhalt einzelne `$name`-Platzhalter
verwendet werden können. `argument-hint` ergänzt das um eine Anzeige in der
Autovervollständigung (z. B. `[environment] [version]`), ersetzt aber keine
der beiden anderen Properties – sie ist rein informativ für den Nutzer.

## Advanced: Dynamische Kontext-Injection

Skills können vor der eigentlichen Ausführung auch Shell-Befehle laufen
lassen und deren Ausgabe direkt in den Prompt einfügen, statt dass Claude
den Befehl erst selbst ausführen muss.

- Die Inline-Syntax `` !`<command>` `` führt den Befehl **sofort aus,
  bevor** Claude den Skill-Inhalt überhaupt sieht. Die Platzhalter-Stelle
  wird durch die tatsächliche Befehlsausgabe ersetzt – Claude bekommt also
  echte Daten statt des Befehls selbst.
- Ein Skill, der z. B. einen Pull Request zusammenfasst, könnte so
  aussehen:

  ```markdown title=".claude/skills/pr-summary/SKILL.md"
  ---
  name: pr-summary
  description: Summarize changes in a pull request
  context: fork
  agent: Explore
  allowed-tools: Bash(gh *)
  ---

  ## Pull request context
  - PR diff: !`gh pr diff`
  - PR comments: !`gh pr view --comments`
  - Changed files: !`gh pr diff --name-only`

  ## Your task
  Summarize this pull request ...
  ```

- Jeder `` !`<command>` ``-Ausdruck wird **einmal** ausgeführt, bevor Claude
  etwas sieht; die Ausgabe wird als reiner Text eingefügt und **nicht**
  erneut nach weiteren Platzhaltern durchsucht. Ein Befehl kann also selbst
  keinen Platzhalter für einen späteren Durchlauf erzeugen.
- Die Inline-Form wird nur erkannt, wenn `!` am Zeilenanfang oder direkt
  nach einem Leerzeichen steht. Steht davor ein anderes Zeichen (z. B.
  `KEY =! 'cmd'`), bleibt der Platzhalter als reiner Text stehen und der
  Befehl wird **nicht** ausgeführt.
- Für mehrzeilige Befehle gibt es eine Block-Variante: einen Codeblock, der
  mit `` ```! `` geöffnet wird:

  ````markdown
  ## Environment
  ```!
  node --version
  npm --version
  git status --short
  ```
  ````

- Über die Einstellung `"disableSkillShellExecution": true` in den Settings
  lässt sich dieses Verhalten für Skills und Custom Commands aus User-,
  Projekt-, Plugin- oder zusätzlichen Verzeichnis-Quellen deaktivieren.
  Jeder Befehl wird dann durch `[shell command execution disabled by
  policy]` ersetzt, statt ausgeführt zu werden. Gebündelte und zentral
  verwaltete ("managed") Skills sind davon nicht betroffen. Am nützlichsten
  ist diese Einstellung in zentral verwalteten Settings, wo Nutzer sie
  nicht selbst überschreiben können.
- Um für einen einzelnen Skill-Lauf tieferes Reasoning anzufordern, reicht
  es, das Wort `ultrathink` irgendwo im Skill-Inhalt zu platzieren.
