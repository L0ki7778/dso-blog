# CLAUDE.md

`CLAUDE.md` ist die Datei, über die dauerhafter Kontext an Claude Code
mitgegeben wird.

- Erstellen lässt sie sich mit `/init` – Claude durchsucht dabei das Projekt
  selbstständig und generiert die Datei automatisch.
- Die Datei muss im **Root**-Verzeichnis liegen.
- Da `CLAUDE.md` immer im Context vorhanden ist, sollte sie **nicht zu groß**
  werden. Wie viel Context aktuell belegt ist, lässt sich mit `/context`
  prüfen.
- Wichtig: Es wird immer die `CLAUDE.md` aus dem Verzeichnis verwendet, in
  dem Claude gestartet wurde – z. B. verwendet `cd monorepo && claude` die
  `CLAUDE.md` aus `monorepo`.
