# Design: git-cliff-migration

## Overview

Replace git-chglog with git-cliff across the release tasks and remove changie,
making git-cliff the single owner of changelog generation. Two artifacts
change: `.taskfiles/task_release.yaml` (task commands) and a new root-level
`cliff.toml` (git-cliff configuration).

## Architecture

```
Taskfile.yaml (includes release: .taskfiles/task_release.yaml)
  └── .taskfiles/task_release.yaml
        env: TAG_PREFIX=v, CHANGES_DIR=.changes, CHANGELOG_FILE=CHANGELOG.md
        tasks:
          prepare         → brew install git-cliff
          batch_changelog → per-tag fragments (loop, --strip all)
          changelog       → single-tag fragment (--strip all)
          changelog_next  → unreleased fragment (--strip all)
          commit_release  → stage CHANGELOG.md + package.json
          create          → git-cliff prepend/output + commit_release
  └── cliff.toml (git-cliff config: conventional commits, grouping, header)
```

### Data flow

- **Fragments** (`.changes/vX.Y.Z.md`): standalone conveniences; content = one
  `## vX.Y.Z - <date>` section with grouped commit bullets. Not consumed by
  `release:create` anymore.
- **CHANGELOG.md**: generated/updated by `release:create` only.
  - Exists → `--prepend` inserts the new release section below `# Changelog`.
  - Missing → `--output` creates header + first release section.

## Key design decisions

| # | Decision | Rationale | Alternatives rejected |
|---|----------|-----------|----------------------|
| 1 | Install via `brew install git-cliff` | git-cliff is not go-installable; brew is simplest on the maintainer's macOS; mise also works but brew is the portable default for the task | npm/cargo/pinned GitHub binary — more moving parts; npm version drift |
| 2 | `filter_unconventional = true` | Verified repo history: 12/13 commits conventional; only "Initial commit" is non-conventional and safe to drop | `filter_unconventional = false` — would surface non-conventional commits under "Other" |
| 3 | Always emit `A..B` ranges | git-cliff 2.14.2 rejects a single rev (`SetCommitRangeError`); first-tag case ranges from `git rev-list --max-parents=0 <tag>` (root commit excluded — it is filtered anyway) | `git-cliff <tag>` — unsupported |
| 4 | Fragments use `--strip all` | `cliff.toml` header (`# Changelog`) must appear in full CHANGELOG.md but must not leak into per-tag fragments | Separate config file for fragments — duplication |
| 5 | `create` = conditional `--prepend` / `--output` | `--prepend` is the git-cliff-native analog of the old `changie merge` incremental update and preserves prior content; `--output` covers first release | Full regen `git-cliff -o CHANGELOG.md` in create — works only for already-tagged history; the not-yet-tagged version would render as "Unreleased" |
| 6 | `commit_release` no longer stages `.changes/` | Fragments are no longer consumed by the release flow; staging an absent directory would fail `git add` | Guarded `git add` — unnecessary once fragments leave the flow |
| 7 | `chore(release)` commits skipped via commit_parsers | Release commits must not appear in changelog sections | n/a |

## Configuration notes (cliff.toml)

- `render_always = true` — a release with no new commits still renders its
  (empty) section, matching `--tag` on unchanged HEAD.
- `sort_commits = "oldest"` — chronological order within sections.
- `protect_breaking_commits = true` — breaking changes survive filtering.
- Group mapping mirrors git-chglog conventions (feat → Features, fix → Bug
  Fixes, chore/build → Miscellaneous Chores, plus perf/refactor/docs/test/
  style/revert/ci).

## Testing strategy

- Verified in a throwaway clone (`git clone` + temporary `v0.1.0`/`v0.2.0`/
  `v0.3.0` tags) to avoid mutating real refs.
- Scenarios: first release, incremental prepend (correct insertion point and
  isolation), full regen, per-tag fragment isolation, fragment header
  stripping, non-conventional commit filtering.
- `release:create` was exercised at the git-cliff command level; the
  `commit_release` step is known-failing due to the pre-existing
  `package.json` issue (out of scope, see requirements.md).

## Security / safety considerations

- No secrets or network calls beyond tool installation.
- `batch_changelog` deletes `.changes/v*.md` before regenerating — scoped to
  the fragments directory only.
