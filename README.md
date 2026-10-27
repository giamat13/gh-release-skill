# gh-release

A Claude Code skill that generates a GitHub release process for a repository. It writes the release instructions, a live local release dashboard with local-test tooling, and release/test workflows if the repository does not have them yet.

The skill only **creates** the instructions and tooling. It never publishes a release itself.

## What it generates

1. **Release instructions**: a `## Releasing` section in the project's `CLAUDE.md` (created if missing, appended to otherwise), and an `AGENTS.md` that points to `CLAUDE.md`. For a program that users install or run, the section also says to add an automatic update system.
2. **Release dashboard**: a local web page plus scripts that run the local tests (stopping at the first failure) and watch the GitHub release workflow. Copied to `tools/release_status/`.
3. **Workflows** (only if the repository has none): a test workflow and a release workflow under `.github/workflows/`. For a Windows program, the release also builds an installer EXE and a portable EXE.

Existing files are extended, never overwritten.

## Requirements

- A GitHub repository with a configured `origin` remote
- Git, with a GitHub token available through `git credential fill`
- Python 3 for the dashboard scripts
- The GitHub CLI (`gh`) is optional

## Install

Copy this folder to your Claude Code skills directory:

- **All projects**: `~/.claude/skills/gh-release/` (on Windows: `%USERPROFILE%\.claude\skills\gh-release\`)
- **One project only**: `<your-project>/.claude/skills/gh-release/`

## Use

In a Claude Code session inside the target repository, ask for it in plain words, for example:

> Set up releases for this repository.

or invoke it directly with `/gh-release`. The skill reads the repository first, asks only what it cannot detect (version numbering, pre-releases, release-notes location and similar), and then generates the files.

## Files

| Path | Purpose |
|---|---|
| `SKILL.md` | The skill procedure Claude follows |
| `references/discovery.md` | What to detect in the repository, and how |
| `references/decisions.md` | The decisions to settle with the user, with recommended defaults |
| `assets/templates/` | Templates for the generated instructions and workflows |
| `assets/dashboard/` | Dashboard and local-test tooling copied into the target repository |

## License

MIT. See [LICENSE](LICENSE).
