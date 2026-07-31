---
name: docs-freshness-check
description: Check whether existing docs/knowledge-base pages (or leftover notes-to-docs source notes) still describe reality accurately, and flag or fix what's drifted. Use this whenever the user asks if their documentation is up to date, out of date, stale, still accurate, or asks you to "check the docs against the real thing" — for both repo-local facts (npm scripts, config keys, folder/category structure) and claims about external tools the docs describe (Claude Code CLI features, model names, Docusaurus behavior, npm package versions).
model: sonnet
effort: medium
---

# Docs Freshness Check

Finds documentation claims that no longer match reality and reports them (or fixes them,
if asked to). This is a **verification** skill, not an authoring one — when it's time to
actually write a fix, follow `.claude/authoring-playbook.md` for the file-format rules
(don't re-derive them here).

## Two kinds of claim, two ways to check them

Every factual claim in this repo's docs falls into one of two buckets. Sort claims into
the right bucket before trying to verify them — checking a local claim against the web (or
vice versa) wastes effort and risks a wrong verdict.

1. **Local/repo-checkable** — claims about *this project*: npm scripts (`package.json`),
   env vars and config keys (`docusaurus.config.ts`, `example.env`), folder/category
   structure (`_category_.yaml` files, `sidebars.ts`), file paths mentioned in the docs.
   Verify these by directly reading the actual file — this is fast, certain, and needs no
   external lookup.
2. **External/product-checkable** — claims about *tools the docs describe*: Claude Code
   CLI behavior (`/model`, `/effort`, skills, permissions), specific model names or context
   window sizes, Docusaurus framework behavior, an npm package's current API. These can
   drift as the external product evolves, independent of anything in this repo. Verify
   these with WebSearch/WebFetch against the authoritative source (e.g. docs.claude.com
   for Claude Code, docusaurus.io for Docusaurus, the npm registry page for a package).

   No MCP connection is needed for this — WebSearch/WebFetch are already available and
   cover the common case. Only reach for a dedicated MCP connector if the user later wants
   to check against something WebSearch/WebFetch genuinely can't reach (e.g. a private
   internal API with no public docs) — don't set one up speculatively.

Some claims aren't checkable at all (opinions, "recommended" framing, "as of writing"
caveats already present in the text) — leave those alone. Only flag concrete, falsifiable
statements.

## Workflow

### 1. Scope the check

`$ARGUMENTS`, if given, names a topic or file to check (e.g. `claude-code` or
`docs/knowledge-base/claude-code/02-models.md`). With no argument, check every page under
`docs/knowledge-base/`.

If the topic's original `notes-to-docs`-style source folder still exists (check for a
sibling folder like `notes-to-docs/` at the repo root), also compare the generated pages
against it — notes edited after the docs were generated are a cheap, certain source of
drift, and don't need any external lookup at all.

### 2. Extract and classify claims

Read each in-scope page and pull out concrete, checkable statements. Classify each as
local or external per the buckets above. Skip prose that isn't a verifiable claim.

### 3. Verify

- **Local claims**: read the actual file (`package.json`, `docusaurus.config.ts`,
  `sidebars.ts`, the relevant `_category_.yaml`, etc.) and compare directly.
- **External claims**: search for the current, authoritative statement and compare. If a
  search comes back genuinely inconclusive (conflicting sources, no clear authoritative
  page), don't guess — report the claim as "unverified" rather than as confirmed or
  contradicted either way. A false "this is outdated" is as costly as missing a real one.

### 4. Report or fix

Default to **reporting**, not silently rewriting — a doc that says something wrong
because the tool changed is a different problem from a doc that was always wrong, and the
user should see the diff before it's applied. For each finding, give:

- File and the specific claim.
- What's actually true now, and the source you checked it against (a file path for local
  claims, a URL for external ones).
- A proposed fix.

Only apply the fix directly, following the playbook's conventions, if the user's request
clearly asked for that (e.g. "find and fix any outdated docs") rather than just "check."
If you do apply fixes, verify afterward:

```bash
npm run build       # fails on any broken internal link (onBrokenLinks: 'throw')
npm run typecheck
```

(`pnpm` may not be on `PATH` in every shell this repo is opened in — fall back to `npm run
build` / `npm run typecheck`, which invoke the identical underlying scripts.)

### 5. Summarize

List what was checked, what's stale (with evidence), what's confirmed current, and
anything left unverified because no reliable source was found.
