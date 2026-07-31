---
name: notes-to-docs
description: Turn a folder of messy, unsorted course notes (markdown) plus related screenshots/images into properly structured documentation pages under docs/knowledge-base/, following this repo's .claude/authoring-playbook.md conventions exactly. Use this whenever the user has a scratch folder of raw notes and images they want sorted and turned into knowledge-base docs, says things like "sort these notes into docs", "add this to the knowledge base", "turn my notes into documentation", or pastes/references a dump of course material with screenshots — even if they don't explicitly ask for a "skill" or describe the folder structure themselves.
model: sonnet
effort: low

---

# Notes → Knowledge Base

Turns a messy folder of notes and screenshots into clean pages under
`docs/knowledge-base/`. This skill is the **orchestration layer** on top of
`.claude/authoring-playbook.md` — that file has the actual file-format rules
(numeric prefixes, `_category_.yaml` shape, the screenshot transcribe-vs-embed
decision). **Read it now if you haven't** — everything below assumes you're
following it. This skill's job is the parts the playbook doesn't cover: reading
an unsorted input folder, matching images to the notes they illustrate, and
deciding where the result belongs.

> **Recommended execution settings: model = Opus, effort = high.**
> If you're running this interactively, set them first (`/model opus`,
> `/effort high`); if you're an orchestrating agent spawning this as a
> subagent/task, request Opus at high effort explicitly. Reason: every step
> here is a judgment call with a real cost if wrong — matching an image to
> the right note passage, telling a genuine near-duplicate apart from two
> related-but-distinct screenshots, and deciding whether new content belongs
> in an existing knowledge-base topic or needs its own. A confidently wrong
> call silently pollutes the user's real documentation tree, which is more
> expensive to notice and unwind than the extra cost of the stronger model.
> A cheaper model is fine for narrow, unambiguous batches (topic argument
> given, one image, one obvious match) — but don't assume that's the common
> case.

## Arguments

Invoked with `args` = `<notes-folder-path> [existing-topic-name]`.

- **One token** (just a folder path) → auto-detect placement (see §4).
- **Two tokens** (folder path + topic name) → the topic name must be an
  **existing** folder under `docs/knowledge-base/<topic-name>/`. Extend it.
  If that folder does **not** exist, stop and tell the user — don't guess or
  auto-create. Either they meant a different existing topic (typo) or they
  want a new one, in which case they should re-run without the second
  argument and let auto-detect create it. Guessing here risks silently
  filing content under the wrong topic.

## Workflow

### 1. Read everything in the notes folder

List the folder's contents. Read every `.md`/`.mdx` file in full. For every
image (`.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.svg`), use the Read tool
directly on the image file — it renders visually, so look at what it actually
shows rather than guessing from the filename.

If the folder doesn't exist or has no markdown files in it, stop and tell the
user rather than inventing content.

### 2. Match images to the notes they illustrate

For each image, decide which part of the notes it belongs to, based on
subject matter (not filename). A screenshot of an operator-precedence table
belongs next to the notes paragraph discussing operators, regardless of
whether the note text and the image were in the same file or even mentioned
each other explicitly.

Two things to watch for, because course-note dumps are messy by nature:

- **Near-duplicate images** (e.g. the same table screenshotted in light and
  dark theme, or the same diagram re-exported) — these should collapse into
  a single transcription/embed. Don't produce two pages describing the same
  table twice.
- **Orphan images** — an image with no matching note content. Don't silently
  drop it or silently invent a place for it: mention it in your final report
  and either fold it in where it plausibly fits or ask the user.

### 3. Sort the notes into a logical structure

Raw notes are rarely in a useful order. Regroup them:

1. Cluster note fragments (and their matched images) by subtopic, ignoring
   the order they appeared in the source file(s).
2. Sequence the clusters in a learning-appropriate order — fundamentals
   before advanced material.
3. Each cluster becomes one page. This is the input to §5 of the playbook
   (numeric-prefixed pages); don't re-derive those file-naming rules here,
   just apply them.

### 4. Decide where it goes

- **Topic argument given** → validate `docs/knowledge-base/<topic>/` exists,
  then extend it: new pages get the next free `NN-` prefix after the highest
  one already in that folder.
- **No topic argument** → read the existing `docs/knowledge-base/*/`
  folders (their overview pages and content) and judge whether the sorted
  note clusters clearly belong to one of them.
  - **Clear match** → extend that folder, same as above.
  - **No clear match** → create a new topic folder. Don't force unrelated
    content into an existing topic just to avoid creating a new one — a
    wrong home is worse than a new folder.
  - If it's genuinely ambiguous between two existing topics, say so in your
    final report and explain which you picked and why, so the user can
    redirect you if you guessed wrong.

### 5. Write the pages

Follow `.claude/authoring-playbook.md` exactly for:

- `_category_.yaml` shape and picking the next free `position` (only when
  creating a new topic folder).
- `00-overview.md` shape (only when creating a new topic folder).
- Minimal frontmatter on content pages.
- The screenshot decision tree — transcribe tables/code/text into real
  Markdown; embed only genuinely visual images (diagrams, UI), co-located in
  `./img/` next to the page.
- Never touching `sidebars.ts`.

### 6. Verify

```bash
npm run build       # fails on any broken internal link (onBrokenLinks: 'throw')
npm run typecheck
```

(`pnpm` may not be on `PATH` in every shell this repo is opened in — `npm run
build` / `npm run typecheck` invoke the identical underlying scripts, so fall
back to those if `pnpm` isn't found.)

Fix anything the build reports before considering the task done.

### 7. Report back

Tell the user, concisely:

- Which topic folder was created or extended, and why (if it wasn't a
  simple "you gave me the argument" case).
- Which pages were added.
- Which images were transcribed vs. embedded vs. left as orphans needing a
  decision.
- That the source notes folder was **not** deleted — it's safe for them to
  clean it up once they're happy with the result.
