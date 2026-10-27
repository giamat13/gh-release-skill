---
name: gh-release
description: Generate a project's GitHub release process - a written release procedure (RELEASING.md, a CLAUDE.md section, or a project skill), a live release dashboard with local-test tooling, and release/test workflows if missing. Use when the user wants to set up releases for a repo, create release instructions, add a release dashboard, or document how to release. This skill only creates the instructions and tooling; it does not perform a release.
---

# gh-release (generator)

This skill **creates** release instructions and release tooling for a project. It does not publish anything. The output is a set of files in the user's repository that a later session (or a person) follows to release.

What gets generated:

1. **Release instructions**: a `## Releasing` section in the project's `CLAUDE.md` describing exactly how to release this project (versioning, notes, tests, push, dispatch, watching, closing issues), plus an `AGENTS.md` that points to `CLAUDE.md`.
2. **Release dashboard**: a live local web page plus scripts that run local tests, stop at the first failure, and watch the GitHub release workflow.
3. **Workflows** (only if the repo has none): a test workflow and a release workflow.

Files in this skill:

- `references/decisions.md` - the decisions to settle with the user, with the options for each. Read before asking questions.
- `references/discovery.md` - what to detect in the repo, and how.
- `assets/templates/release-instructions.md` - the template for the generated instructions (placeholders are `{{LIKE_THIS}}`).
- `assets/templates/tests.yml`, `assets/templates/release.yml` - generic workflow templates.
- `assets/dashboard/` - the dashboard and test tooling to copy and adapt: `status_page.py` (it writes `status.html` itself), `run_local_tests.py`, `watch_tests.py`, `watch_release.py`, `log.py`, `reset.py`.

## Procedure

### Step 1: Discover

Follow `references/discovery.md`. Collect: repository `owner/name` (from `git remote`), default branch, where the version number lives, existing tags and their format, existing workflows, how tests are run, where release notes live (if anywhere), what the build produces. Do not ask the user anything you can read from the repo.

### Step 2: Settle the decisions

Follow `references/decisions.md`. Ask only what discovery could not answer, in one `AskUserQuestion` round when possible. Propose a recommended default for each question.

### Step 3: Generate the instructions

1. Put the instructions in the target repo's **`CLAUDE.md`**, as a `## Releasing` section built from `assets/templates/release-instructions.md`:
   - If `CLAUDE.md` does not exist, create it with that section.
   - If it exists, add the section at the end (or update an existing release section). Never overwrite or reword the rest of the file.
2. Create **`AGENTS.md`** in the target repo root that points to Claude's file:
   ```
   # AGENTS.md

   All instructions for this repository are in [CLAUDE.md](CLAUDE.md). Read it first and follow it.
   ```
   If `AGENTS.md` already exists, do not overwrite it: add that one line at the top instead.
3. Replace every `{{PLACEHOLDER}}` with the discovered or decided value. Delete the sections and options that do not apply (for example pre-releases, if the user does not want them); do not leave placeholders, "if applicable" hints, or options the project does not use.
4. Every command in the result must be real for this project (actual test command, actual version file, actual workflow file name, actual default branch).
5. Write it in the language the user asked for; keep file names, commands and code as they are.

### Step 4: Install the dashboard

1. Copy `assets/dashboard/*` to `tools/release_status/` (or the folder the user chose).
2. Adapt it:
   - `status_page.py`: `VERSION_FILE` and `VERSION_RE` (where the version number lives and a regex for it), `RELEASE_WORKFLOW` (the `name:` of the release workflow), and the `JOBS`/`STEPS` dictionaries (the project's workflow job and step names, each with a one-line plain description). The repository and default branch are read from the `origin` remote automatically.
   - `run_local_tests.py`: `DEFAULT` (the project's test commands or names) and `command()` (how one test is started).
   - `watch_release.py` and `watch_tests.py`: normally need no change.
3. Add the generated files to `.gitignore`: `status.html`, `status-data.js`, `local-tests*.json`, `local-single.json`, `activity.json`, `cycle.json`, `archive/` (adjust if the page is kept elsewhere).
4. The GitHub token comes from `git credential fill`; make sure nothing writes it to a file or prints it.

### Step 5: Workflows

Only if the repo has no release workflow: copy `assets/templates/release.yml` and `tests.yml` to `.github/workflows/`, fill in the real build and test commands, and keep this structure: the release workflow is started by `workflow_dispatch`, runs the tests as a reusable workflow in parallel with the builds, and its `publish` job `needs` the tests and builds so nothing is published if they fail. If the repo already has workflows, adapt the instructions and dashboard to them instead of replacing them.

### Step 6: Verify what you generated

- Run `python tools/release_status/status_page.py --once` and confirm `status-data.js` appears and the page opens with no errors.
- Run one real local test through `run_local_tests.py <name>` and confirm the dashboard card shows it.
- Re-read the generated instructions as someone who has never seen the project: every command must be runnable as written.
- Do not dispatch a release, push a tag, or publish anything while verifying.

### Step 7: Report

Tell the user which files were created, how to open the dashboard, and what the instructions say a release looks like, in a few lines.
