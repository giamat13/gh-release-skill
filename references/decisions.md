# Decisions to settle with the user

Ask only what discovery (`discovery.md`) could not answer. Give a recommended default for each.

## 1. Where do the instructions live?

Fixed, not a question: a `## Releasing` section in the target repo's `CLAUDE.md`, and an `AGENTS.md` that points to `CLAUDE.md`. Existing files are extended, never overwritten.

## 2. Version numbering

Pick one and write it down exactly, including which number rises when.

- **Semantic (MAJOR.MINOR.PATCH)**: breaking change / new feature / fix. Recommended for libraries.
- **FEATURE.SMALL-FEATURE.FIX**: first number for a feature, second for only small features, third for only fixes. Only ONE number rises per release (the biggest change in it) and the ones after it reset.
- **Calendar (YYYY.MM.PATCH)**.
- Always write all three parts if tools downstream parse versions.

Also settle: the single source of truth for the version (one file), the tag format (`vX.Y.Z`), and whether a commit that lands after a release forces a new version (recommended: yes).

## 3. Pre-releases

- None.
- Pre-release OF a regular version plus a counter: tag `vX.Y.Z-beta.N`, title `X.Y.Z (pre N)`, counter restarts at 1 for each new version, the version file stays plain, pre-releases stay as history afterwards. Settle when one is allowed (for example only for a huge change, or for something that can only be verified in a built and installed program).

## 4. Release notes

- Location (`docs/release-notes-vX.Y.Z.md` recommended; the file becomes the GitHub Release body) or generated from commits/PRs.
- Sections, in order, omitting empty ones, for example: `Features`, `Small features`, `Fixes`. They should match the kind of version number that rose.

## 5. Branch and merge policy

- Release from the default branch directly (no feature branches), or from a release branch, or via pull requests. Write the real policy.

## 6. Pre-release checks

- Which local tests must pass (the list), and whether they run in parallel.
- Whether the change must also be checked for real (running the program, calling real endpoints, driving the real UI) before any release; recommended: yes, tests alone are not enough.
- Whether a failed test stops the run at once (recommended) and is then re-run alone before it is called broken.
- Anything else that must be true before releasing (docs updated, translations, lock files, migrations).

## 7. What a release produces

- The assets expected in the GitHub Release (installer, archives, packages) and their count, so the procedure can verify them.
- For a Windows program: an installer EXE and a portable EXE (a single file that runs without installing). Recommended: build both in the release job on `windows-latest`, with the installer tool the project already has (Inno Setup, NSIS, or the packager's own installer target) and a single-file build for the portable one (PyInstaller `--onefile`, electron-builder `portable`, `dotnet publish` as one file). Name them with the version, for example `app-{version}-setup.exe` and `app-{version}-portable.exe`, and list both among the expected assets.
- Signing, if any, and which secrets/variables the workflow needs (never committed; written to an untracked file during the build).

## 8. Issues

- Whether fixed issues are commented with the version and closed after the release (recommended), and who may close them.

## 9. Recovering from a bad release

- Delete the release and its tag (`gh release delete vX.Y.Z --cleanup-tag -y`) and release again, or ship a new patch version. Write which.

## 10. Dashboard

- Open it through VS Code Live Server on the repository root, or as a local file.
- Language of the page text.
- Whether the local tests also show a popup on failure (off by default).

## 11. Automatic updates

Ask only when the project ships a program that users install or run: an installer, an executable, a desktop or mobile app. A library, or a package that a package manager updates (pip, npm, NuGet, cargo), has no update system to add: delete the `## Automatic updates` section.

- When it checks: at start-up in the background, at most once a day (recommended), or only from a menu item.
- Which updater: a library for the stack when there is one (for example `electron-updater` for Electron, `tauri-plugin-updater` for Tauri, Velopack or Squirrel for .NET), otherwise a small checker that reads `releases/latest` and downloads the asset.
- Checksum: the file is installed only when its SHA-256 matches the value published with the release (recommended, the minimum). Code signing, if the project already has it, is a stronger check on top.
- Asset name: the pattern the updater looks for (for example `app-{version}-setup.exe`). The release workflow must produce exactly that name and publish the checksum file. The updater installs the installer EXE from section 7; the program does not update the portable EXE.
- Visibility: the updater reads the release without a token, so the repository must be public. For a private repository the updater cannot read releases without a token, and a token must not ship inside the program. Tell the user this first, and ask where the assets should be published.
