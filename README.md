# Résumé

LaTeX source for my résumé (`resume.tex`). The built PDF is `Daniel_Zhang_Resume.pdf`, which is also served at [zhangdz.ca](https://zhangdz.ca).

## How it stays up to date

- **Build:** every push that changes `resume.tex` runs a GitHub Action that compiles it, fails if the PDF is longer than one page, and commits `Daniel_Zhang_Resume.pdf` and `resume-preview.webp` back to that branch.
- **Sync:** a weekly Claude Code routine (Sundays, 6 a.m. Toronto) reads [`SYNC.md`](SYNC.md), checks every repo listed in [`sync-state.json`](sync-state.json) (plus any new public repo) for commits since the last review, and opens a `resume-sync` pull request with any résumé changes those commits justify. Each change is listed with the commit that supports it.
- **Website:** when `main` here has a PDF the website doesn't, the routine opens an `Update résumé PDF` pull request in the `website` repo.

Nothing reaches `main` here or on the site without me merging a PR. The one exception is `sync-state.json`, which the routine updates on its own when it found nothing to change.

To stop watching a repo, move it from `repos` to `ignore` in `sync-state.json`.
