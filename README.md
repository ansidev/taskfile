# taskfile

ansidev's frequently used [Taskfiles](https://taskfile.dev/).

## Manual

See all available tasks:

```
task --list
```

## Task commands

| Command                   | Action                                                            |
| ------------------------- | ----------------------------------------------------------------- |
| `dep:commit`              | Commit changes after updating package version                     |
| `dep:init_dep_branch`     | Init git branch deps/*                                            |
| `dep:update`              | Check whether the new version of the npm package broke the build. |
| `git:push_current`        | Push current branch                                               |
| `release:batch_changelog` | Generate changelog for existing git tags                          |
| `release:changelog`       | Generate changelog for an existing version                        |
| `release:changelog_next`  | Generate changelog for a new version                              |
| `release:commit_release`  | Create release commit                                             |
| `release:create`          | Generate release commit                                           |
| `release:prepare`         | Install tools                                                     |
| `site:build`              | Build site                                                        |
| `site:clean`              | Clean the build output                                            |
| `site:install`            | Install dependencies                                              |

## Contact

Le Minh Tri [@ansidev](https://ansidev.xyz/about).

## License

This source code is available under the [MIT License](/LICENSE).
