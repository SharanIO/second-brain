# Git cleanup runbook — remove large PDFs from history

**Why this exists:** `NLP.pdf` (~211 MB) was committed to git history. GitHub rejects any push containing a file >100 MB. `.gitignore` only blocks future tracking — it does not retroactively remove the file from past commits. This runbook rewrites history to drop those files entirely, then force-pushes the clean history.

**Risk level:** This rewrites git history. If you have the vault cloned on another machine, you will need to re-clone there after this. If anyone else has cloned the repo (unlikely for a personal vault), they'll need to re-clone too. There is no in-place "pull" recovery from a force-push of rewritten history.

**Estimated time:** 10–15 minutes including the backup.

---

## Step 0 — Back up the repo

Before doing anything destructive, make a complete copy. Run from outside the vault:

```powershell
# PowerShell
Compress-Archive -Path "D:\second-brain" -DestinationPath "D:\second-brain-backup-$(Get-Date -Format yyyyMMdd-HHmm).zip"
```

```bash
# Git Bash equivalent
cp -r /d/second-brain /d/second-brain-backup-$(date +%Y%m%d-%H%M)
```

If anything goes wrong below, you can restore from this backup.

---

## Step 1 — Confirm the file is in history

```bash
cd /d/second-brain   # or `cd D:\second-brain` in PowerShell
git log --all --oneline -- "20-concepts/NLP.pdf" "raw/incoming/NLP.pdf"
```

If this returns commits, the file is in history and the cleanup is needed. If it returns nothing, the file was never committed and `.gitignore` alone is sufficient — you can stop here.

You may also want a broader audit of what's bloating the repo:

```bash
# Top 10 largest objects in git history
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | grep '^blob' | sort -k3 -n -r | head -10
```

If this surfaces other binaries you didn't expect, add them to the filter-repo step below.

---

## Step 2 — Install `git-filter-repo`

This is the modern, fast tool for rewriting history. It's a Python script.

```bash
pip install git-filter-repo
```

Verify:

```bash
git filter-repo --version
```

If `pip` isn't available, install Python first (`winget install Python.Python.3.12` works on Windows).

---

## Step 3 — Note your remote URL

Filter-repo strips the remote as a safety measure. Capture it now so you can re-add it.

```bash
git remote -v
```

Copy the `origin` URL (something like `https://github.com/<you>/second-brain.git` or `git@github.com:<you>/second-brain.git`). Save it somewhere — clipboard, sticky note, whatever.

---

## Step 4 — Rewrite history to remove the large files

```bash
git filter-repo --path 20-concepts/NLP.pdf --invert-paths
git filter-repo --path raw/incoming/NLP.pdf --invert-paths
```

If your audit in Step 1 surfaced additional files, add a `--path <file> --invert-paths` line for each.

After this, `git log` will show the same commits but the file blobs are gone from every commit.

Verify:

```bash
git log --all --oneline -- "20-concepts/NLP.pdf" "raw/incoming/NLP.pdf"
```

This should now return **nothing**.

---

## Step 5 — Re-add the remote

```bash
git remote add origin <your-github-url-from-step-3>
```

---

## Step 6 — Force-push the cleaned history

```bash
git push --force-with-lease origin main
```

(Replace `main` with `master` or whatever your default branch is — check with `git branch --show-current`.)

`--force-with-lease` is safer than `--force` because it refuses if the remote has commits you don't have locally. Since this is a personal repo, that's unlikely, but it's good hygiene.

If GitHub still complains about size, the bloat in the pack file may be cached. Run:

```bash
git gc --prune=now --aggressive
git push --force-with-lease origin main
```

---

## Step 7 — Verify on GitHub

Go to the GitHub web UI → your repo → check that `NLP.pdf` doesn't appear anywhere in the file tree, and check the repo's "Storage" usage (Settings → General → near the bottom). It should drop by ~210 MB after GitHub's garbage collection runs (may take a few minutes).

---

## Step 8 — The PDF on disk

The actual `NLP.pdf` file is still on your local disk in `20-concepts/` and `raw/incoming/`. The new `.gitignore` excludes all `*.pdf`, so they're safe — they just stop being tracked.

If you want them out of those wiki folders entirely (cleaner), move them to a non-vault location:

```bash
mkdir -p ~/local-references
mv "20-concepts/NLP.pdf" ~/local-references/
mv "raw/incoming/NLP.pdf" ~/local-references/
```

Adjust `~/local-references/` to wherever you want — somewhere outside `D:\second-brain` so it never re-enters the vault.

---

## Step 9 — Sanity-check Obsidian Git plugin behavior

The Obsidian Git plugin makes `vault-auto:` commits. If those commits are still failing to push, double-check:

- Plugin Settings → "Pull updates on startup": fine to leave on.
- Plugin Settings → "Disable push": should be **off**.
- Plugin Settings → commit/push interval: increase HTTP timeout if needed (in your global git config, not the plugin):

  ```bash
  git config --global http.postBuffer 524288000
  ```

---

## After this is done

Going forward, `*.pdf` (and other binary patterns) are gitignored. Drop new papers into `raw/papers/` or `raw/incoming/` freely — they stay local, your markdown wiki integration is what gets pushed.

If you ever want to actually version-control a specific small PDF (rare), use `git add -f path/to/file.pdf` to bypass `.gitignore` for that one file.
