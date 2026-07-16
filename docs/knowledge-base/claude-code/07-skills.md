# Skills

Skills erweitern Claude Code um wiederverwendbare Fähigkeiten – vergleichbar
mit einem zusätzlichen, fest hinterlegten Prompt, der bei Bedarf automatisch
mitgeliefert wird.

## Skills erstellen

- Neue Skills werden mit dem `/skill-creator`-Befehl in der Claude CLI
  erstellt.
- Ein Skill braucht mindestens einen **Titel** (`name`) und eine
  **Description**. Alle weiteren Properties sind optional – siehe die
  vollständige Übersicht unten.
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
`SKILL.md`, zwischen den beiden `---`-Zeilen:

| Property | Pflicht? | Beschreibung |
|---|---|---|
| `name` | Pflicht | Eindeutiger Skill-Name; sollte mit dem Ordnernamen übereinstimmen. |
| `description` | Pflicht | Entscheidet, ob und wann der Skill automatisch vom Model verwendet wird (siehe oben). Sollte auch den Auslöse-Kontext klar benennen. |
| `allowed-tools` | optional | Schränkt ein, welche Tools der Skill verwenden darf, z. B. erlaubt `Bash(gh *)` nur `gh`-Aufrufe über Bash. |
| `context` | optional | z. B. `fork` – lässt den Skill in einem eigenen, abgetrennten Kontext laufen statt direkt im Hauptgespräch. |
| `agent` | optional | Legt fest, welcher Subagent-Typ den Skill ausführt, z. B. `Explore`. |

Die Kursnotizen erwähnten außerdem `model`, `effort` und eine Property für
"User- oder Model-Invocation". Ein direkter Test (Skill mit `model: haiku`,
ohne `context: fork`) zeigte keinerlei Wirkung – der Skill lud identisch zu
einem Skill ganz ohne diese Properties, ohne erkennbaren Model-Wechsel oder
Subagent-Aufruf. Da sie sich nicht bestätigen ließen, stehen sie hier
absichtlich nicht in der Tabelle.

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
