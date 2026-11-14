# Tasks: git-cliff-migration

> As-built record: implementation exists in the working tree on branch
> `TFS-1` and was verified on 2026-11-14 (git-cliff 2.14.2). Checkbox states
> reflect completed implementation pending final commit/review.

- [x] 1. Add `cliff.toml` at repository root
  - [x] 1.1 Conventional-commit parsing, grouping, `chore(release)` skip, `filter_unconventional = true`, `protect_breaking_commits = true`, `render_always = true`, `sort_commits = "oldest"`
  - [x] 1.2 Header `# Changelog`, empty footer, per-version body template (`## vX.Y.Z - <date>` + grouped bullets)

- [x] 2. Update `.taskfiles/task_release.yaml` env
  - [x] 2.1 Add `CHANGELOG_FILE: CHANGELOG.md` alongside `TAG_PREFIX`, `CHANGES_DIR`

- [x] 3. Replace git-chglog fragment tasks with git-cliff
  - [x] 3.1 `release:changelog` — `git-cliff --strip all "<prev>..<tag>" --output .changes/<tag>.md`; prev tag via versionsort + awk exact match; first-tag fallback to root commit
  - [x] 3.2 `release:changelog_next` — `git-cliff --strip all --unreleased --tag <tag> --output .changes/<tag>.md`
  - [x] 3.3 `release:batch_changelog` — same tag loop with the range logic from 3.1
  - [x] 3.4 `mkdir -p ${CHANGES_DIR}` in all three fragment tasks

- [x] 4. Remove changie, give git-cliff ownership of CHANGELOG.md
  - [x] 4.1 `release:create` — `--prepend CHANGELOG.md` when the file exists, `--output` otherwise (both `--unreleased --tag <tag>`)
  - [x] 4.2 `release:commit_release` — stage `CHANGELOG.md` + version file only
  - [x] 4.3 `release:prepare` — drop changie (and git-chglog) installs

- [x] 5. Verification (throwaway clone, throwaway tags)
  - [x] 5.1 First release: header + `## v0.1.0` section from all commits; "Initial commit" filtered
  - [x] 5.2 Incremental prepend: new section inserted below header, contains only new commits
  - [x] 5.3 Full regen `git-cliff -o CHANGELOG.md`: header + all tagged sections, newest first
  - [x] 5.4 Fragments: per-range isolation confirmed; no `# Changelog` leakage
  - [x] 5.5 Regression caught and fixed during verification: prev-tag resolution originally picked the newest tag overall; now picks the tag immediately preceding the target

- [ ] 6. Wrap-up
  - [ ] 6.1 Commit changes (`feat(task:release): replace git-chglog with git-cliff; drop changie`) — committed 2026-11-15, branch `TFS-1` pushed
  - [x] 6.2 Spec approved by user via kiro-impl preflight (2026-11-15); `spec.json` approvals set to `approved: true`
  - [ ] 6.3 Move kanpal issue TFS-1 to done — deferred until PR merge per user decision (2026-11-15); branch committed and pushed
  - [x] 6.4 `package.json` staging bug: decided out of scope (user, 2026-11-15); logged as kanpal follow-up issue TFS-2

## Implementation Notes

- The migration implementation (tasks 1–5) predates the spec; it was implemented directly from kanpal issue TFS-1 and later documented as an as-built spec.
- git-cliff 2.14.2 rejects a single rev (`SetCommitRangeError`) — always emit `A..B` ranges; first tag falls back to the repo root commit (excluded from output; non-conventional and filtered anyway).
- The `.changes/` fragment dir may not exist on first run; tasks now `mkdir -p` before generating.
- Pre-existing `package.json` staging bug is tracked as TFS-2, not fixed here.
