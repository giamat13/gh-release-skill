## Releasing {{PROJECT_NAME}}

Repository: `{{REPO}}`. Default branch: `{{DEFAULT_BRANCH}}`. Version number lives in `{{VERSION_FILE}}` (the single source of truth). Tag format: `{{TAG_FORMAT}}`. Release workflow: `.github/workflows/{{RELEASE_WORKFLOW}}`.

## Rules

1. **Never release before the change is known to work.** {{REAL_CHECK_RULE}}
2. {{BRANCH_POLICY}}
3. Never release on top of a CI run you already know is red.
4. {{BAD_RELEASE_RULE}}
5. The GitHub token always comes from `git credential fill` and is never printed or written to a file.

## Version numbers

{{VERSIONING_SCHEME}}

{{PRERELEASE_SECTION}}

## Release notes

File: `{{NOTES_PATH}}` (it becomes the GitHub Release body). Format:

{{NOTES_FORMAT}}

## Automatic updates

The program updates itself from this repository's GitHub releases. If it does not do that yet, add it as a normal change: tests first, then a release.

- Updater: {{UPDATER}}.
- It checks {{CHECK_RULE}}. It asks `https://api.github.com/repos/{{REPO}}/releases/latest` without a token and compares that tag with the version it runs. Pre-releases are never offered.
- It downloads only the asset named `{{ASSET_PATTERN}}` and installs it only when the file's SHA-256 matches its line in `{{CHECKSUM_FILE}}`, which is published in the same release.
- After step 10 of the procedure, check that `releases/latest` returns the new tag and that the assets have the names above. A renamed asset or a missing checksum file stops updates for every installed copy.

## Progress view

Progress is shown in the agent-dashboard: https://github.com/giamat13/agent-dashboard. It is set up separately; this repository does not generate a page.

- Keep it current: update its task list at every step, and show each long command (tests, build, release watch) in it, as its README explains.

## Local tests

`{{TEST_COMMAND}}` runs the whole set (`{{TEST_COUNT}}` suites) and writes `local-tests.json`, which `watch_tests.py` reads. `{{TEST_COMMAND}} <names>` runs only those; a single name is shown in its own card.

- {{PARALLEL_RULE}}
- Pass nothing but test names or `--parallel N` / `--serial`: any other word is run as a test name and shows up as a failed test.
- Every new test is added to the `DEFAULT` list in `run_local_tests.py` **and** to the CI test workflow.
- **Stop at the first failure.** Run `{{WATCH_TESTS_COMMAND}}` in the background: it deletes a stale `local-tests.json`, starts the run, reads the progress every 3 seconds and, at the first failed test, kills the run, prints the failure and exits 1; when everything passed it prints `FINISHED OK` and exits 0. Fix, then run everything again from the start.
- A test that failed in the full run is run again alone before it is called broken (UI tests can fail under load).
- Never start a second run while one is running and never restart one that is running (shared ports and progress file). Before the next run make sure no test process or server is left over.
{{EXTRA_TEST_RULES}}

## Procedure

1. Update the agent-dashboard: its task list and the current step.
2. Collect what goes in: open issues and `git log <last-tag>..HEAD --oneline`. Decide the version number.
3. Bump `{{VERSION_FILE}}` and write the release notes.
4. {{REAL_CHECK_STEP}}
5. Run the local tests with the watcher (above).
6. Commit and push to `{{DEFAULT_BRANCH}}`; wait until `git status -sb` shows no "ahead".
7. Check the tag does not exist: `GET /repos/{{REPO}}/git/ref/tags/<tag>` must be 404.
8. Dispatch: `POST /repos/{{REPO}}/actions/workflows/{{RELEASE_WORKFLOW}}/dispatches` with `{"ref":"{{DEFAULT_BRANCH}}","inputs":{"prerelease":"false"}}` (204). Do not release by pushing a tag; the workflow reads the version, creates the tag, builds and publishes. {{TESTS_IN_RELEASE_NOTE}}
9. Watch it: `{{WATCH_RELEASE_COMMAND}}` in the background (prints the jobs on change, exits 1 at the first failed job, 0 on success). At the first failure: read the job log (`GET /actions/jobs/<id>/logs`), cancel the run (`POST /actions/runs/<id>/cancel`), fix, push, dispatch again.
10. On success check that the Release has all its assets ({{EXPECTED_ASSETS}}).
11. {{ISSUES_STEP}}
12. Finish the agent-dashboard: mark every task done.

Typical durations: {{TYPICAL_DURATIONS}}

## Recovering from a bad release

{{RECOVERY}}

## Common problems

| Symptom | Action |
|---|---|
| Tests fail on a busy port | A test process or server is left over: close it, run again |
| `git push` of a tag fails | Do not push tags; dispatch the workflow |
| API returns 401 | `git credential fill` returned an empty password: sign in to git again |
{{EXTRA_PROBLEMS}}
