# Skills

Skills erweitern Claude Code um wiederverwendbare Fähigkeiten – vergleichbar
mit einem zusätzlichen, fest hinterlegten Prompt, der bei Bedarf automatisch
mitgeliefert wird.

## Skills erstellen

- Neue Skills werden mit dem `/skill-creator`-Befehl in der Claude CLI
  erstellt.
- Ein Skill braucht mindestens einen **Titel** (`name`) und eine
  **Description**. Optional lassen sich außerdem festlegen: ein bestimmtes
  **Model**, **Permissions**, ein **Effort**-Level sowie ob der Skill vom
  Nutzer oder vom Model selbst ausgelöst werden darf.
- Skills liegen unter `.claude/skills/<skill-name>/SKILL.md`. Der
  Ordnername (`<skill-name>`) sollte mit dem `name` des Skills
  übereinstimmen.
- Ob und wann ein Skill selbstständig vom Harness verwendet wird, hängt
  maßgeblich von einer guten **Description** ab: Passt die aktuelle
  Situation zur Description, greift Claude auf den Skill zurück.
- Nach dem Erstellen eines neuen Skills muss der Client neu geladen werden
  (`/reload`).
- Weil Skills ein eigenes Model festlegen können, lassen sich verschiedene
  Modelle nutzen, **ohne den Prompt-Cache zu brechen** – anders als ein
  manueller `/model`-Wechsel mitten in der Session.

## Argumente

Skills können Argumente entgegennehmen. Im Skill werden sie über
`$ARGUMENTS` referenziert:

```yaml title=".claude/skills/deploy/SKILL.md"
---
name: deploy
description: Deploys our codebase to either staging or production
model: sonnet
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
