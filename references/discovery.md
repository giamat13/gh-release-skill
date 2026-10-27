# Discovery: what to detect in the repo

Read the repo; do not ask what it can tell you. Run commands in the shell the platform uses.

| What | How to find it |
|---|---|
| Repository `owner/name` | `git remote get-url origin` |
| Default branch | `git symbolic-ref refs/remotes/origin/HEAD` or `git branch --show-current` |
| Version number location | Search for the version string: `pyproject.toml`, `package.json`, `Cargo.toml`, `*.csproj`, `version.py`, `setup.cfg`, `pom.xml`, `build.gradle`, a `VERSION` file. The file the build reads is the source of truth. |
| Tag format | `git tag --sort=-creatordate \| head`: `v1.2.3` or `1.2.3`, any `-beta.N` / `-rc.N` pre-release tags |
| Existing releases | `gh release list --limit 5` (or `GET /repos/{repo}/releases`) |
| Existing workflows | `.github/workflows/*.yml`: triggers (`push`, `workflow_dispatch`, `workflow_call`), job names, what `publish` needs |
| How tests run | `package.json` scripts, `pytest`/`tox`/`Makefile` targets, `tools/test_*.py`, CI test job steps |
| Tests with fixed ports or shared state | Search test files for fixed ports / shared temp folders; they cannot run in parallel |
| Release notes | A `docs/release-notes-*`, `CHANGELOG.md`, or none |
| Build output | Workflow `upload-artifact` / release `files:` entries; the packaging spec files |
| Secrets/variables used by workflows | `${{ secrets.X }}` and `${{ vars.X }}` in the workflows (list the names, never values) |
| Existing instructions | `CLAUDE.md`, `RELEASING.md`, `CONTRIBUTING.md`, `docs/` - reuse them, do not contradict them |
| Token availability | `git credential fill` (check only that a password is returned; never print it) |
| Platform/shell | Windows PowerShell vs bash: write commands in the shell the user actually uses |

Also note: whether the repo is public or private, whether `gh` is installed, and whether the Python (or Node) used by the dashboard scripts is available (`py -3 --version` / `python3 --version`).
