# Documentation Authoring Playbook

Guidance for turning **pasted notes and screenshots** into documentation pages in this
Docusaurus repo. Follow it exactly — the sidebar is generated from the folder tree, so
file names, folders, and `_category_` files *are* the configuration. Getting them right
matters more than anything else.

A complete worked example that follows every rule below lives at
`docs/knowledge-base/python/` — copy its shape when in doubt.

---

## 1. When to use this

Trigger: the user pastes raw notes and/or screenshots and wants them turned into docs.
Your job: **sort** the material into a sensible structure and write pages that match the
conventions here. Ask the user which topic the notes belong to only if it is unclear.

## 2. Where content lives

All docs are under `docs/`, split into three areas:

- `docs/knowledge-base/` — reference/learning material, grouped one folder per subject
  (`git/`, `Container/`, `DevOps/`, `React/`, `Angular/`, `shells/`, `python/`). **This is
  the default home for course notes.**
- `docs/guides/` — how-to guides (Docusaurus-tutorial-derived).
- `docs/projects/` — project write-ups.

### Placement decision

1. Does a folder for this subject already exist under `docs/knowledge-base/`?
   - **Yes** → add a new page to it with the next free `NN-` prefix (see §3).
   - **No** → create a new topic folder `docs/knowledge-base/<topic>/` (see §5).
2. One subject = one folder. One subtopic = one page inside it.

## 3. Ordering: numeric filename prefixes (knowledge-base rule)

In `docs/knowledge-base/`, page order in the sidebar comes from a **numeric filename
prefix**, not from frontmatter:

```
00-overview.md
01-operators.md
02-datatypes.md
```

Docusaurus strips the `NN-` prefix from the URL. Start each topic with `00-overview.md`.
Do **not** use `sidebar_position` frontmatter here (that convention is only used in
`docs/guides/`).

## 4. Frontmatter: keep it minimal

Most pages need little or none. Use the lightest form that works:

- Content page: usually **no frontmatter** — just start with a `#` heading, which becomes
  the title and sidebar label.
- Overview page referenced by a category link (§5): `title: Overview`.
- Override the sidebar label only when needed: `sidebar_label: Short Name`.

Full form (`id`, `slug`, `title`, `description`) exists in a few pages but is not required.

## 5. Creating a new topic folder

Create `docs/knowledge-base/<topic>/` with:

**`_category_.yaml`** — use `.yaml` (the dominant extension). Pick the **next free
`position`** (existing: shells=1, git=2, Container=3, DevOps=4, React=5, Angular=5,
python=6 → next new topic = 7). Two link styles:

Link to an overview page (preferred — gives the category a landing page):
```yaml
label: "Topic Name"
link:
    type: "doc"
    id: knowledge-base/<topic>/overview
position: 7
```

Or an auto-generated index (no landing page needed):
```yaml
label: "Topic Name"
link:
    type: "generated-index"
position: 7
```

> If you use `type: "doc"`, the `id` must resolve to a real page. `00-overview.md` with
> `title: Overview` resolves to `knowledge-base/<topic>/overview`. Get this wrong and
> `pnpm build` fails (broken links throw — see §8).

**`00-overview.md`** — landing page:
```md
---
title: Overview
---

One or two sentences introducing the topic.

import DocCardList from '@theme/DocCardList';

<DocCardList/>
```

**`01-<subtopic>.md`, `02-…`** — the actual content pages.

## 6. Screenshots: transcribe first, embed only when necessary

This is the most important judgment call. A screenshot is not searchable, not
theme-aware (looks wrong in the other color mode), and not accessible.

**Decision tree for every pasted screenshot:**

- Screenshot of a **table** → rewrite it as a Markdown table. (See
  `docs/knowledge-base/python/01-operators.md` — built from a table screenshot.)
- Screenshot of **code / terminal output** → rewrite it as a fenced code block with a
  language tag (```` ```python ````, ```` ```bash ````, etc.).
- Screenshot of **plain text** → transcribe it as text.
- Screenshot of a **diagram, chart, UI, or anything genuinely visual** → embed the image
  (only case where you keep the image).

**Embedding an image (visual content only):** co-locate it in an `./img/` subfolder next
to the markdown file and reference it with plain Markdown:

```
docs/knowledge-base/<topic>/
  img/
    architecture-diagram.png
  02-architecture.md   →   ![Architecture diagram](./img/architecture-diagram.png)
```

- Use Markdown `![alt](./img/name.png)` syntax only — **no** MDX `import` of images, **no**
  `<img>` tags.
- Name assets descriptively in lowercase-kebab-case.
- Always write meaningful alt text.

## 7. Sorting notes into pages

Raw notes are usually an unordered brain-dump. Turn them into structure:

1. **Group** the notes by subtopic.
2. **One page per subtopic**, sequenced with `NN-` prefixes in a logical learning order
   (fundamentals before advanced).
3. Within a page, use a heading hierarchy: one `#` title, `##` sections, `###`
   subsections. Open with a one-sentence intro, then details.
4. Convert every code sample to a fenced code block **with a language tag**.
5. **Preserve the source language** of the notes (German notes → German pages; this repo
   already mixes German and English).
6. Tighten wording, fix obvious typos — but don't invent facts that aren't in the notes.

> **Behavioral claims can go stale.** Notes and screenshots capture what the author observed
> at one point in time. A claim about how a tool behaves ("switching X does not break the
> cache", "Y needs no confirmation") is only as good as that observation — it's not a
> permanent guarantee, and the tool can change. If the user later reports a live
> contradiction (they tried it and got a different result), trust their lived experience
> over the originally transcribed note and correct the page — don't defend the transcription
> just because it matches the source material. `.claude/skills/docs-freshness-check/SKILL.md`
> exists for systematically re-checking existing pages against current reality; reach for it
> when a page makes several such claims or hasn't been touched in a while.
>
> **Tested-false beats hedged-unconfirmed.** If a claim gets actively tested (not just
> doubted) and the test contradicts it, remove the claim outright rather than leaving it in
> the page marked "unconfirmed" — a hedge is for something genuinely untested or
> unverifiable, not for something that already failed a real test. Leaving disproven content
> in place, even caveated, is worse than deleting it: it still reads as documentation.

## 8. Hard rules checklist

- [ ] Knowledge-base ordering uses `NN-` filename prefixes, **not** `sidebar_position`.
- [ ] Category files are `_category_.yaml`, with a unique `position`.
- [ ] `type: "doc"` category links point at a page that actually exists.
- [ ] Frontmatter is minimal.
- [ ] Tables/code/text from screenshots are **transcribed**, not embedded.
- [ ] Embedded images (visual only) sit in a co-located `./img/` and use Markdown syntax.
- [ ] **Never edit `sidebars.ts`** — the sidebar is autogenerated from the folder tree.
- [ ] All internal links resolve (broken links fail the build — see below).

## 9. Finish & verify

```bash
pnpm typecheck        # TypeScript check
pnpm build            # also fails on ANY broken internal link (onBrokenLinks: 'throw')
pnpm start            # optional: eyeball the new pages in light AND dark mode
```

Fix everything `pnpm build` reports before committing. Then commit with a
[Conventional Commit](../docs/knowledge-base/git/03-conventional-commits.md) message,
e.g.:

```
docs: add <topic> notes to the knowledge base
```
