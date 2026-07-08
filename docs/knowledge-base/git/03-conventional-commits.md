---
id: conventional-commits
title: Conventional Commit Messages
sidebar_label: Conventional Commits
---

# Conventional Commit Messages

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) is a lightweight convention for writing commit messages. It gives you an easy way to explain **why** a change was made and lets tools automatically generate changelogs, determine semantic version bumps, and keep commit history consistent across a team.

## Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Commit Types

| Type       | Used When                                                        | Example                                      |
|------------|-------------------------------------------------------------------|-----------------------------------------------|
| `feat`     | Introducing a new feature                                        | `feat: added calculator feature`             |
| `fix`      | Fixing a bug                                                     | `fix: correct division by zero in calculator` |
| `docs`     | Changing documentation only                                      | `docs: update README with setup instructions` |
| `style`    | Code style changes with no logic change (formatting, whitespace) | `style: format code with prettier`           |
| `refactor` | Restructuring code without changing behavior or fixing a bug     | `refactor: simplify calculator input parsing` |
| `perf`     | Improving performance                                            | `perf: optimize matrix multiplication`       |
| `test`     | Adding or correcting tests                                       | `test: add unit tests for calculator`        |
| `build`    | Changes to the build system or external dependencies             | `build: upgrade webpack to v5`               |
| `ci`       | Changes to CI configuration or scripts                           | `ci: add GitHub Actions workflow for tests`  |
| `chore`    | Other changes that don't modify src or test files                | `chore: update .gitignore`                   |
| `revert`   | Reverting a previous commit                                      | `revert: revert "feat: added calculator feature"` |

## Breaking Changes

A commit that introduces a breaking API change should either append a `!` after the type/scope, or include a `BREAKING CHANGE:` footer.

```bash
feat!: remove support for Node 12

feat: change calculator API

BREAKING CHANGE: `calculate()` now returns a Promise instead of a value
```

## Scopes

An optional scope can be added in parentheses to clarify which part of the codebase the commit affects:

```bash
feat(calculator): added support for exponents
fix(auth): handle expired tokens correctly
```
