---
name: describe-to-docs
description: Create a new docs/knowledge-base page or edit an existing one directly from a plain-language description of what the user wants written — no notes file or screenshots required. Use this whenever the user describes documentation content in their own words and wants it written or an existing page changed, e.g. "add a page about pnpm workspaces to the git section", "update the operators page to mention modulo with negative numbers", "rewrite the Python overview intro" — even without a notes-to-docs folder of raw notes and images. If the user does hand over an optional notes file or a screenshot alongside the description, incorporate it too.
model: opus
effort: medium
---

# Describe → Docs

Writes or edits a single `docs/knowledge-base/` page straight from a description, instead
of from a folder of raw notes. This skill is the **orchestration layer** on top of
`.claude/authoring-playbook.md` — that file has the actual file-format rules (numeric
prefixes, `_category_.yaml` shape, the screenshot transcribe-vs-embed decision). **Read it
now if you haven't.** This skill's job is the part the playbook doesn't cover: figuring out
whether the description means "new page," "new topic," or "edit this existing page," from
a description alone rather than a pre-sorted batch of notes.

If the user instead hands over a folder of unsorted notes and screenshots to turn into
docs, that's `.claude/skills/notes-to-docs/SKILL.md`, not this one.

## Input

`$ARGUMENTS` is the description itself — freeform text. It may optionally reference a
notes file or image path inline (e.g. "using the screenshot at notes-to-docs/foo.png"); if
so, read that file too and fold it in per the playbook's transcribe-vs-embed rule (§6) —
don't re-derive those rules here.

## Workflow

### 1. Figure out what's being asked

Read the description for two signals:

- **Edit or create?** Phrases like "update," "fix," "rewrite," "also cover X in the Y
  page" point at editing something that already exists. "Add a page about," "document,"
  "write up" without naming an existing page point at creating new content.
- **Where?** The description may name a topic or page explicitly ("in the git section,"
  "the operators page"). If so, that's your target — confirm it actually exists before
  editing it.

### 2. Locate the target

- If the description names an existing page/topic, verify it with Glob/Grep before
  touching anything — don't assume a name matches without checking.
- If it's ambiguous which existing topic (or page within one) is meant, or whether this
  should be a new topic vs. added to one that already exists, **ask the user** rather than
  guessing. Unlike notes-to-docs (which processes a whole batch unattended), this skill
  runs on a single, interactive request — a quick clarifying question costs less than
  writing to the wrong place.
- If it's clearly new content with no existing match, follow the playbook's placement
  decision (§2): one subject = one folder, next free `NN-` prefix or next free
  `_category_` `position` for a brand-new topic.

### 3. Write or edit the content

- **New page**: follow the playbook's frontmatter (§4), ordering (§3), and — if this is
  also a new topic folder — the `_category_.yaml` and `00-overview.md` templates (§5).
- **Edit**: make the smallest change that satisfies the description. Match the voice and
  language of the surrounding page (this repo mixes German and English by section — don't
  switch a German page to English mid-edit).
- If a notes file or image was referenced in the description, apply the playbook's
  transcribe-vs-embed decision tree (§6) to it before writing.
- Don't invent facts the description didn't give you. If the description is too thin to
  write an accurate page (e.g. it names a topic but gives no actual content), ask for more
  detail rather than filling gaps with guesses.

### 4. Verify

```bash
npm run build       # fails on any broken internal link (onBrokenLinks: 'throw')
npm run typecheck
```

(`pnpm` may not be on `PATH` in every shell this repo is opened in — `npm run build` /
`npm run typecheck` invoke the identical underlying scripts, so fall back to those if
`pnpm` isn't found.)

### 5. Report back

Tell the user, concisely: which file was created or edited, where it landed (and why, if
placement wasn't obvious from the description), and confirm the build/typecheck passed.
