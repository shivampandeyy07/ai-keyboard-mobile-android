Nice — here’s a clear, copy-pasteable **step-by-step git workflow** that keeps your branded work separate, lets you safely pull updates from FlorisBoard, and *doesn’t collapse your changes* with upstream changes.

# Setup (run once in your repo)

1. Go to your repo folder:

```bash
cd ai-keyboard-mobile-android
```

2. Verify remotes:

```bash
git remote -v
```

You should see `origin` (your fork). If you haven't already added `upstream`, add it now:

```bash
git remote add upstream https://github.com/florisboard/florisboard.git
git fetch upstream
```

3. Create branch structure:

```bash
# create a development branch for your work
git checkout -b dev

# push dev to your origin and set tracking
git push -u origin dev

# create a local branch that mirrors upstream/main (keeps upstream changes isolated)
git fetch upstream
git checkout -B upstream-main upstream/main
# (optional) push upstream-main to your origin as a backup
git push -u origin upstream-main
```

4. Decide / create a production branch:

* If you want `main` to be your production branch, skip below.
* If you want a separate `prod` branch:

```bash
git checkout main        # make sure main is checked out and up-to-date
git checkout -b prod
git push -u origin prod
```

# Daily / regular workflow to pull updates from FlorisBoard without collapsing your changes

**A. Bring upstream changes into your local `upstream-main`**

```bash
# fetch latest upstream
git fetch upstream

# reset local upstream-main to match upstream/main exactly
git checkout upstream-main
git reset --hard upstream/main
# (safer alternative if you don't want hard reset)
# git checkout upstream-main
# git pull --ff-only upstream main
```

(`reset --hard` ensures `upstream-main` exactly mirrors FlorisBoard. Use with care — upstream-main is just a mirror branch.)

**B. Inspect upstream commits (optional)**

```bash
git log upstream-main --oneline -n 20
```

**C. Merge selected upstream changes into your dev branch (non-destructive)**

```bash
git checkout dev
git merge --no-ff upstream-main   # creates a merge commit, keeps histories separate
# resolve conflicts if any:
#  - edit conflicted files
#  - git add <file1> <file2>
#  - git commit
git push origin dev
```

* `merge --no-ff` preserves both histories and avoids collapsing your commits into upstream’s.
* If you prefer a linear history, you can `git rebase upstream-main` onto dev, but **rebasing rewrites history** — avoid on public branches already pushed/shared.

**D. Cherry-pick single commits from upstream (if you only want specific fixes)**

```bash
# find commit hash from upstream-main:
git log upstream-main --oneline

# cherry-pick into your dev:
git checkout dev
git cherry-pick <commit-hash>
# resolve conflicts if any, then:
git push origin dev
```

Use cherry-pick when you want only one or a few upstream commits, keeping everything else out.

# When dev is ready for production

```bash
# test thoroughly, then merge dev into prod (or main)
git checkout prod   # or `main` if that's your production branch
git merge --no-ff dev
# resolve conflicts, if any
git push origin prod
```

Or create a Pull Request from `dev` → `prod/main` on GitHub so CI checks run and you get a review step.

# Conflict resolution quick commands

During a merge, if conflicts occur:

```bash
git status               # shows conflicted files
# edit files to resolve conflicts
git add <file1> <file2>
git commit               # completes the merge
# or if in rebase:
# git rebase --continue
```

# Safety & best practices (short)

* **Do not `git reset --hard` on `dev` or `main`** unless you're 100% sure. Use reset only on `upstream-main`.
* Use **feature branches** for each new AI feature: `feature/ai-autocomplete`, `feature/ui-branding`. Work there, then merge into `dev`.
* **Protect** your production branch (`main` or `prod`) in GitHub (branch protection rules) so changes must go through a PR and checks.
* Prefer `git merge upstream-main` to bring upstream changes into your dev — it preserves both histories and won’t “collapse” your commits.
* Use `cherry-pick` to selectively bring single commits from upstream.

# Ready-to-run minimal command sequence (copy-paste)

```bash
cd ai-keyboard-mobile-android
git remote add upstream https://github.com/florisboard/florisboard.git   # skip if already added
git fetch upstream
git checkout -b dev
git push -u origin dev
git checkout -B upstream-main upstream/main
git push -u origin upstream-main   # optional backup
```

# TL;DR

* Keep an **`upstream-main`** branch that mirrors FlorisBoard.
* Do your work on **`dev`** and feature branches.
* Merge (or cherry-pick) from `upstream-main` into `dev` when you want updates — this keeps your changes separate and under your control.
* Merge `dev` into `prod/main` only when you’re ready for release.

If you want, I can generate a small bash script with these commands (and safe checks) that you can run locally to automate this workflow. Want that?





Not exactly 🙂

Let me explain carefully:

```bash
git remote add upstream https://github.com/florisboard/florisboard.git
```

* This **does NOT create a branch**.
* It just adds a **remote pointer** called `upstream` that points to FlorisBoard’s repo.
* After this, Git knows there’s another source of truth besides your own fork (`origin`).

---

### How to actually get FlorisBoard’s branch locally

1. Fetch upstream’s branches:

```bash
git fetch upstream
```

Now you’ll see remote-tracking branches like:

```
upstream/main
upstream/dev
```

2. If you want a **local branch** that tracks FlorisBoard’s main:

```bash
git checkout -b upstream-main upstream/main
```

This creates a **local branch** `upstream-main` that starts from FlorisBoard’s `main`.

---

### ✅ So the sequence is:

```bash
git remote add upstream https://github.com/florisboard/florisboard.git
git fetch upstream
git checkout -b upstream-main upstream/main
```

Now you have:

* `origin/main` → your fork’s main (your code)
* `upstream/main` → FlorisBoard’s main (read-only remote tracking branch)
* `upstream-main` → your **local branch** mirroring FlorisBoard’s main (you control it)

---

⚡ This is the safe setup you’ll use to **pull FlorisBoard updates** without overwriting your branded changes.

Do you want me to give you a **diagram of how these remotes/branches connect** so it’s crystal clear?




----


Nice — here’s a **clear, no-confusion summary + exact step-by-step commands** you can copy-paste. I’ll keep it short and practical.

# Summary (one line)

Keep your work on `dev`/`feature/*`, keep a local mirror branch `upstream-main` that follows FlorisBoard, and selectively merge or cherry-pick upstream commits into your `dev` so your changes never get collapsed.

---

# Initial setup (run once)

```bash
cd ai-keyboard-mobile-android

# add upstream (only if you haven't already)
git remote add upstream https://github.com/florisboard/florisboard.git
git fetch upstream

# create a development branch for your work
git checkout -b dev
git push -u origin dev

# create a local branch that mirrors upstream/main
git checkout -b upstream-main upstream/main

# optional: push upstream-main to your origin as a backup
git push -u origin upstream-main
```

> Note: `git remote add upstream ...` **does NOT** create a branch. It only registers the remote. `git fetch upstream` creates remote-tracking refs like `upstream/main`. `git checkout -b upstream-main upstream/main` creates the local `upstream-main` branch.

---

# Regular update workflow (when you want FlorisBoard updates)

1. Fetch upstream and update your local mirror:

```bash
git fetch upstream
git checkout upstream-main
# safer: fast-forward only (recommended)
git pull --ff-only upstream main
# or (if you want exact mirror and are OK overwriting upstream-main):
# git reset --hard upstream/main
```

2. Inspect upstream changes (optional):

```bash
git log upstream-main --oneline -n 20
```

3. Merge upstream into your dev (keeps histories separate — **recommended**):

```bash
git checkout dev
git merge --no-ff upstream-main
# resolve conflicts if any
git push origin dev
```

4. Or cherry-pick a single upstream commit you want:

```bash
git checkout dev
git cherry-pick <commit-hash>
# resolve conflicts if any, then:
git push origin dev
```

---

# When `dev` is ready for production

```bash
# test well, then merge to your production branch (main or prod)
git checkout main        # or `prod`
git merge --no-ff dev
git push origin main
```

(Or open a PR dev → main on GitHub to use CI/review.)

---

# Quick conflict resolution

During a merge:

```bash
git status                # shows conflicted files
# edit files to fix conflicts
git add <file1> <file2>
git commit                # finishes the merge
git push origin dev
```

If rebasing: `git rebase --continue` after resolving each conflict.

---

# Best practices (short)

* Work in feature branches: `feature/ai-autocomplete`, `feature/ui-branding`.
* Keep `main` (or `prod`) protected and merge via PRs.
* Use `merge --no-ff` to preserve both histories (prevents collapsing).
* Use `cherry-pick` to pull only specific upstream commits.
* Use `upstream-main` purely as a mirror of FlorisBoard; don’t develop on it.

---

# Minimal copy-paste sequences

**Initial**

```bash
cd ai-keyboard-mobile-android
git remote add upstream https://github.com/florisboard/florisboard.git
git fetch upstream
git checkout -b dev
git push -u origin dev
git checkout -b upstream-main upstream/main
git push -u origin upstream-main
```

**Update from FlorisBoard → merge into dev**

```bash
git fetch upstream
git checkout upstream-main
git pull --ff-only upstream main
git checkout dev
git merge --no-ff upstream-main
git push origin dev
```

---

If you want, I can turn these commands into a tiny bash script you run locally to automate the update steps. Want that?



.......


Nice — here’s a **clear, no-confusion summary + exact step-by-step commands** you can copy-paste. I’ll keep it short and practical.

# Summary (one line)

Keep your work on `dev`/`feature/*`, keep a local mirror branch `upstream-main` that follows FlorisBoard, and selectively merge or cherry-pick upstream commits into your `dev` so your changes never get collapsed.

---

# Initial setup (run once)

```bash
cd ai-keyboard-mobile-android

# add upstream (only if you haven't already)
git remote add upstream https://github.com/florisboard/florisboard.git
git fetch upstream

# create a development branch for your work
git checkout -b dev
git push -u origin dev

# create a local branch that mirrors upstream/main
git checkout -b upstream-main upstream/main

# optional: push upstream-main to your origin as a backup
git push -u origin upstream-main
```

> Note: `git remote add upstream ...` **does NOT** create a branch. It only registers the remote. `git fetch upstream` creates remote-tracking refs like `upstream/main`. `git checkout -b upstream-main upstream/main` creates the local `upstream-main` branch.

---

# Regular update workflow (when you want FlorisBoard updates)

1. Fetch upstream and update your local mirror:

```bash
git fetch upstream
git checkout upstream-main
# safer: fast-forward only (recommended)
git pull --ff-only upstream main
# or (if you want exact mirror and are OK overwriting upstream-main):
# git reset --hard upstream/main
```

2. Inspect upstream changes (optional):

```bash
git log upstream-main --oneline -n 20
```

3. Merge upstream into your dev (keeps histories separate — **recommended**):

```bash
git checkout dev
git merge --no-ff upstream-main
# resolve conflicts if any
git push origin dev
```

4. Or cherry-pick a single upstream commit you want:

```bash
git checkout dev
git cherry-pick <commit-hash>
# resolve conflicts if any, then:
git push origin dev
```

---

# When `dev` is ready for production

```bash
# test well, then merge to your production branch (main or prod)
git checkout main        # or `prod`
git merge --no-ff dev
git push origin main
```

(Or open a PR dev → main on GitHub to use CI/review.)

---

# Quick conflict resolution

During a merge:

```bash
git status                # shows conflicted files
# edit files to fix conflicts
git add <file1> <file2>
git commit                # finishes the merge
git push origin dev
```

If rebasing: `git rebase --continue` after resolving each conflict.

---

# Best practices (short)

* Work in feature branches: `feature/ai-autocomplete`, `feature/ui-branding`.
* Keep `main` (or `prod`) protected and merge via PRs.
* Use `merge --no-ff` to preserve both histories (prevents collapsing).
* Use `cherry-pick` to pull only specific upstream commits.
* Use `upstream-main` purely as a mirror of FlorisBoard; don’t develop on it.

---

# Minimal copy-paste sequences

**Initial**

```bash
cd ai-keyboard-mobile-android
git remote add upstream https://github.com/florisboard/florisboard.git
git fetch upstream
git checkout -b dev
git push -u origin dev
git checkout -b upstream-main upstream/main
git push -u origin upstream-main
```

**Update from FlorisBoard → merge into dev**

```bash
git fetch upstream
git checkout upstream-main
git pull --ff-only upstream main
git checkout dev
git merge --no-ff upstream-main
git push origin dev
```

---

If you want, I can turn these commands into a tiny bash script you run locally to automate the update steps. Want that?
