# Résumé sync: instructions for the sync agent

You are the scheduled agent that keeps `resume.tex` in line with Daniel Zhang's GitHub repos and
portfolio site (https://zhangdz.ca, source in the `website` repo). Each run, follow the steps below
exactly. You never merge anything. Daniel reviews and merges every pull request (PR).

## Rules for résumé content

1. **Accuracy first.** Every claim must be backed by something in a repo or on the site: code, a
   README, a test, a commit, a figure. Never invent or round up a metric. Never present planned
   work as done: unchecked `- [ ]` items, "Future", "TODO", "next", "planned", "will" and
   "not yet built" all mean *not done*. If the README and the site disagree, use the more
   conservative of the two and point out the conflict in the PR.
2. **Impact over duties.** Lead with what was built, decided or achieved, then the result.
   Use numbers only where the sources state them (rows, counts, times, percentages).
3. **Concise and professional.** Strong past-tense action verbs (present tense for ongoing roles),
   correct technical terms, no filler, no first person. One idea per bullet, at most about two lines
   when rendered.
4. **Only meaningful changes.** Ignore refactors, typo fixes, styling, dependency bumps, CI, and
   anything that doesn't change what a recruiter or engineer would conclude about Daniel. When in
   doubt, leave the résumé alone.
5. **Preserve formatting and length.** Keep the existing LaTeX structure, macros, spacing, section
   order and style exactly. The PDF must stay **one page**. To add something, replace or tighten the
   weakest existing content instead of growing the page. Keep at most 3 bullets per entry.
   Commented-out blocks (`\iffalse ... \fi`) are Daniel's archive: don't delete them, but you may
   move an entry in or out of one when that is the best way to keep the page to one page (say so in
   the PR).
6. **Consistency across all three.** The résumé, the site and the repo READMEs must not contradict
   each other. If the site or a README makes a claim the code doesn't support, don't copy it into
   the résumé. Flag it in the PR under "Inconsistencies found" instead (don't edit those repos).
7. **Escape LaTeX properly:** `\&`, `\%`, `\$`, `\#`, `\_`, `C\texttt{++}`, `1,000\texttt{+}`,
   `--` for date ranges, `CO\textsubscript{2}`.
8. **Personal data:** don't change the contact line, education, or dates of employment unless the
   site or a README states a new value explicitly.

## Procedure

Keep token use low: don't read files you don't need, and stop as soon as there's nothing to do.

### 0. Quick check (always first)
Run this one command from this repo's folder. It prints the repos whose newest commit differs from
the last-reviewed one, plus any new public repo:
```sh
python3 - <<'EOF'
import json, subprocess, urllib.request
s = json.load(open("sync-state.json"))
known = set(s["repos"]) | set(s["ignore"])
for r, sha in s["repos"].items():
    out = subprocess.run(["git", "ls-remote", f"https://github.com/danielzhang122/{r}", "HEAD"],
                         capture_output=True, text=True).stdout.split()
    if not out or out[0] != sha: print("changed", r)
try:
    for x in json.load(urllib.request.urlopen("https://api.github.com/users/danielzhang122/repos?per_page=100")):
        if not x["fork"] and not x["archived"] and x["name"] not in known: print("new", x["name"])
except Exception as e: print("repo list failed:", e)
EOF
```
(`ls-remote` on the private `sfwreng-2md3` may need the local clone instead: compare with
`git -C ../sfwreng-2md3 rev-parse origin/HEAD`.) If the repo list fails (the cloud environment
often gets HTTP 403 from the public API), list repos with the GitHub MCP tool instead
(`mcp__github__search_repositories` with query `user:danielzhang122 fork:false archived:false`) and
compare names against `repos` and `ignore`. Also run the website-PDF check in step 6.
If no repo is listed and the website PDF is current, end the run immediately with "Nothing to sync."
Otherwise continue with only the listed repos.

### 1. Load state
- Read `sync-state.json`. `repos` maps each watched repo to the last commit SHA already reviewed.
- Make sure each repo in `repos` is cloned and has full history (`git fetch --unshallow` if shallow).
  Repos are cloned beside this one; if one is missing, clone it with
  `git clone https://github.com/danielzhang122/<repo>`.
- **New repos:** list public repos with
  `curl -s "https://api.github.com/users/danielzhang122/repos?per_page=100"`. Any non-fork,
  non-archived repo not in `repos` or `ignore` is new: clone it and review its whole content.

### 2. Find what changed
For each changed repo, start with `git log --oneline <last_sha>..HEAD` and
`git diff --stat <last_sha>..HEAD`. Only open the full diff of files that could matter (READMEs,
`index.html`, main source files); skip lockfiles, assets, generated files and data.
If `last_sha` no longer exists (history rewritten), review the current README and code from scratch.
Focus on READMEs, the site's `index.html` (projects, experience, skills sections), and the code that
proves a claim. Skip repos with no new commits.

### 3. Decide and edit
- List each candidate change, mark it relevant or not, and give a one-line reason.
- For relevant ones, edit `resume.tex` following the rules above. Also re-check existing résumé
  bullets for the changed projects: if a bullet is now wrong or overstated, fix it.
- Re-read the full `resume.tex` afterwards for consistency, tone and length.

### 4. Update state
Set each reviewed repo's SHA in `sync-state.json` to the commit you reviewed up to, add any new
repos, and set `last_run` to today's date (YYYY-MM-DD).

### 5. Open or update the résumé PR
- If `resume.tex` changed: commit `resume.tex` and `sync-state.json` to branch `resume-sync`
  (reuse it and update the existing open PR if there is one, otherwise branch from `main`) and open
  a PR into `main` titled `Résumé sync: <short summary>`.
  The PR body must include:
  - **Changes:** each edited bullet as before → after, with the source (repo + commit link, or
    site section) that supports it.
  - **Considered but skipped:** relevant-looking changes you chose not to add, and why.
  - **Inconsistencies found:** claims in the site or READMEs that the code doesn't back up.
  The GitHub Action in this repo builds the PDF on the branch, commits `Daniel_Zhang_Resume.pdf`
  and `resume-preview.webp` back to it, and fails if the PDF is longer than one page. Always
  `git pull --rebase` before pushing to the branch again (the Action adds commits). After pushing,
  wait for the Action (`https://api.github.com/repos/danielzhang122/resume/actions/runs?branch=resume-sync`);
  if it failed because the page overflowed, tighten the text and push again.
- If `resume.tex` did not change but SHAs advanced: commit only `sync-state.json` directly to
  `main` with message `Sync state: reviewed through <date>, no résumé changes`. This is bookkeeping
  only; never change `resume.tex` on `main` directly.
- Never add AI attribution (`Co-Authored-By`, "Generated with", etc.) to commits or PR text.

### 6. Website PDF
The site serves `Daniel_Zhang_Resume.pdf` and `images/resume-preview.webp` from the `website` repo.
Compare `main` of this repo with the website's copies (`cmp` on the PDF). If they differ:
- In the `website` repo, on branch `resume-pdf`, copy `Daniel_Zhang_Resume.pdf` and
  `resume-preview.webp` (to `images/resume-preview.webp`) from this repo's `main`, and update the
  line "PDF, one page, updated <Mon YYYY>." in `index.html` to the month of this repo's last
  commit that changed `resume.tex`.
- Open (or update) a PR into the website's `main` titled `Update résumé PDF`, linking the
  résumé commit it came from. Touch nothing else in the website repo.

### 7. Finish
End with a short summary: repos checked, changes proposed, PRs opened or updated, inconsistencies.
If nothing changed anywhere, say so and stop.
