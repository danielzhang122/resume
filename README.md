# Résumé

LaTeX source for my résumé (`resume.tex`). The built PDF is `Daniel_Zhang_Resume.pdf`, which is also served at [zhangdz.ca](https://zhangdz.ca).

Dynamically updates to reflect changes across my GitHub repositories and projects.

## How it works

Once a week, a scheduled Claude Code agent runs [`SYNC.md`](SYNC.md):

1. **`scripts/check.py`** (plain Python, no AI) pulls every repo listed in `sync-state.json` and
   lists files changed since the last-reviewed commit, ignoring the pipeline's own outputs. On a
   quiet week it prints `NOTHING TO SYNC` and the run ends there.
2. If something changed, the agent reads only those diffs and decides whether they're résumé-worthy
   (shipped work only, no invented numbers, one page).
3. Edits go on a new `resume-sync-<date>` branch. **GitHub Actions** compiles the LaTeX, fails if
   it's more than one page, and commits the PDF and preview image back.
4. The agent opens a PR listing each change as before → after, with the commit that backs it up.
   I reword and merge it myself. A follow-up PR copies the new PDF to the website.

The design, test results and trade-offs are in [`docs/PIPELINE.md`](docs/PIPELINE.md).

## Use it for your own résumé

Point Claude (Claude Code, or any coding agent) at this repo:

> Set up the résumé sync from https://github.com/danielzhang122/resume for me. Follow its INSTALL.md.

[`INSTALL.md`](INSTALL.md) walks the agent through it: gather your name, GitHub user and résumé repo,
convert your résumé to LaTeX if needed, run [`scripts/install.py`](scripts/install.py) (copies the
code, tests and workflows and fills in [`template/SYNC.md`](template/SYNC.md) for you), verify the
build, and schedule the weekly task. Opening this repo in Claude Code also loads the
`install-resume-sync` skill, which does the same.
