# Reading and Inspecting History

**Skill area:** Git 
**Topic:** `git log`,`git log --oneline`, `git diff`,`git diff --staged` `HEAD~N`
**Session:** 02

---

## The Big Picture

**In plain language:** Git keeps a complete, immutable record of every change ever committed to a repository. Reading that history — and understanding what changed between any two points — is how you orient yourself in an unfamiliar codebase, audit your own work before pushing, and answer "what broke and when" after an incident.

**Why it matters:** In cloud ops and DevOps, you will regularly inherit repos mid-project. The ability to read history and diff state is the difference between spending five minutes understanding a codebase and spending an hour guessing. In production, when something breaks after a deploy, git history is your first forensic tool.

---

## The Concept

### Mental Model

Git history is a chain of immutable receipts. Every commit is a receipt: it has a timestamp, an author, a message, and a pointer to the previous receipt. `git log` is the receipt printer. `git diff` is the magnifying glass that shows exactly what changed between two states of the repository.

### How It Works

**`git log` — the full receipt**

The default output gives you everything: full hash, author name and email, date, and the full commit message. Useful when you need to read commit messages carefully or copy a full hash.

```bash
git log
# Full output per commit: hash, author, date, message
# Verbose — use when you need the full picture
```

**`git log --oneline` — the working view**

One line per commit. Abbreviated hash (7 chars) and the first line of the commit message. This is what you actually use day-to-day when scanning history.

```bash
git log --oneline
# 57dbcc3 (HEAD -> main) Revert "chore: add wrong config"
# 747e3c5 chore: add wrong config — revert candidate
```

**`git log --oneline --graph` — branch visualizer**

Draws the branch structure alongside the history. On a linear single-branch repo, it's just asterisks. The moment you have merges or multiple branches, the graph shows how they converge. Add `--all` to include every branch and remote tracking ref, not just the current one.

```bash
git log --oneline --graph --all --decorate
# * 57dbcc3 (HEAD -> main) Revert "chore: add wrong config"
# * 747e3c5 chore: add wrong config — revert candidate
```

`--decorate` adds ref labels in parentheses — branch names, HEAD position, remote tracking refs. Once a remote exists, you'd see `(HEAD -> main, origin/main)` when they're in sync, or `origin/main` on an older commit when you're ahead. That label difference is instant signal about sync state.

**`git diff` — unstaged changes**

Compares the working directory to the staging area. Shows what has changed but not yet been staged. Once a file is staged, `git diff` goes empty for that file — the working directory and staging area now match.

```bash
git diff
# Shows: working directory vs staging area
# Goes empty after git add — this is correct, not a bug
```

**`git diff --staged` — staged changes**

Compares the staging area to the last commit. Shows exactly what will go into the next commit. Run this before every commit to confirm what you're actually packaging.

```bash
git diff --staged
# Shows: staging area vs last commit
# This is what git commit will record
```

**Reading diff output**

Every diff follows the same format. Lines prefixed with `-` were removed. Lines prefixed with `+` were added. Unchanged context lines have no prefix. No `+` lines means a pure deletion. No `-` lines means a pure addition.

```bash
@@ -1,2 +1 @@
 lab2
-wrong config value
# The line "wrong config value" was removed. Nothing was added.
```

**Comparing specific commits**

Two ways to reference commits in a diff:

```bash
git diff <older-hash> <newer-hash>   # explicit — use when comparing two specific points
git diff HEAD~1 HEAD                  # relative — use when looking back N commits from now
git diff HEAD~5 HEAD                  # "what changed across the last 5 commits"
```

`HEAD` is always the current commit. `HEAD~1` is one commit back. `HEAD~N` is N commits back. Relative notation is faster than copying hashes when you're doing a quick lookback.

**The audit commands**

For inspecting a single commit in full — message plus diff in one output:

```bash
git show <hash>
# commit message + full diff for that commit
```

For reading history with diffs inline:

```bash
git log -p -5
# Last 5 commits, each with its full diff
# Fastest way to audit recent changes in one pass
```

---

## Drills

### Drill 1 — `git log` variants

**What I did:**

```bash
git log
git log --oneline
git log --oneline --graph
```

**Output:**

```
# git log (abbreviated)
commit 57dbcc3a3587e0816c74c7017ceaf97a6d427d36 (HEAD -> main)
Author: LM-10 <lm.10@email.com>
Date:   Sun Jun 14 19:02:15 2026 -0300

    Revert "chore: add wrong config — revert candidate"

# git log --oneline
57dbcc3 (HEAD -> main) Revert "chore: add wrong config — revert candidate"
747e3c5 chore: add wrong config — revert candidate
a73b0b6 Revert "chore: add bad config line"
80408f7 chore: add bad config line
6168de5 chore: add changes to readme for lab2 work
256e84a chore: add a temp lab2.txt file for lab work
8010445 docs(readme): add session 2 section header
5a24011 chore(gitignore): add venv and macos folder attributes to ignore
1868ff4 chore(readme): add description to readme
41e590f chore: add .gitignore
effdd24 docs(readme): initial commit — add README

# git log --oneline --graph
* 57dbcc3 (HEAD -> main) Revert "chore: add wrong config — revert candidate"
* 747e3c5 chore: add wrong config — revert candidate
[... linear, all asterisks]
```

**What this taught me:** `git log` gives full detail per commit — useful when you need to read the full message or copy a complete hash. `--oneline` is what you actually use daily — fast scan of what happened and when. `--graph` earns its place the moment branches diverge or merges happen. On a linear history it's just asterisks, but with multiple branches it draws the convergence visually. The `--all` flag is the important addition — without it, you only see the current branch.

---

### Drill 2 — `git diff` with hashes and `HEAD~` notation

**What I did:**

```bash
git diff 747e3c5 8010445
git diff HEAD~1 HEAD
```

**Output:**

```
# git diff 747e3c5 8010445
diff --git a/README.md b/README.md
index 55916f4..9d0a72b 100644
--- a/README.md
+++ b/README.md
@@ -3,4 +3,3 @@
 This repository contains hands on drills and command I used to learn git step by step.

 ## Session -2
-In this sessions we will be learning about git logs ane reverting changes in in git.
diff --git a/lab2.txt b/lab2.txt
deleted file mode 100644
index f62747c..0000000
--- a/lab2.txt
+++ /dev/null
@@ -1,2 +0,0 @@
-lab2
-wrong config value

# git diff HEAD~1 HEAD
diff --git a/lab2.txt b/lab2.txt
index f62747c..5721d27 100644
--- a/lab2.txt
+++ b/lab2.txt
@@ -1,2 +1 @@
 lab2
-wrong config value
```

**What this taught me:** `HEAD~1` means one commit behind current HEAD — relative navigation without needing to copy hashes. The `~` notation is more practical for quick lookbacks: `HEAD~3` means three commits back from wherever I am now. Hash notation is better when comparing two specific non-adjacent points. Both produce identical output when pointing to the same commits.

---

### Drill 3 — `git diff` vs `git diff --staged`

**What I did:**

```bash
echo "test edit" >> lab2.txt
git diff
git add lab2.txt
git diff
git diff --staged
```

**Output:**

```
# git diff (before staging)
diff --git a/lab2.txt b/lab2.txt
index 5721d27..931cba2 100644
--- a/lab2.txt
+++ b/lab2.txt
@@ -1 +1,2 @@
 lab2
+test edit

# git diff (after staging) — empty

# git diff --staged
diff --git a/lab2.txt b/lab2.txt
index 5721d27..931cba2 100644
--- a/lab2.txt
+++ b/lab2.txt
@@ -1 +1,2 @@
 lab2
+test edit
```

**What this taught me:** `git diff` compares working directory to staging area. After `git add`, those two states match — nothing left to diff, so output is empty. That's correct behavior, not a bug. `git diff --staged` then compares staging area to the last commit, which is where the change now lives. The two commands answer different questions: "what haven't I staged yet" vs "what's about to go into my commit."

---

### Drill 4 — `--decorate` and ref labels

**What I did:**

```bash
git log --oneline --graph --all --decorate
```

**Output:**

```
* 57dbcc3 (HEAD -> main) Revert "chore: add wrong config — revert candidate"
* 747e3c5 chore: add wrong config — revert candidate
* a73b0b6 Revert "chore: add bad config line"
* 80408f7 chore: add bad config line
* 6168de5 chore: add changes to readme for lab2 work
[...]
```

**What this taught me:** `--decorate` adds the ref labels in parentheses — `HEAD -> main` tells me where HEAD is pointing and which branch it's on. Without a remote, there's no `origin/main` label. Once pushed, I'd see `(HEAD -> main, origin/main)` when in sync, or `origin/main` on an older commit when my local branch is ahead. That label placement is how you read sync state at a glance without running `git status` or `git fetch`.

---

## Lab

**Scenario:** You're onboarding onto a repo mid-project. You need to get up to speed on what's changed recently and audit the state of the working tree before touching anything. Produce a written audit note suitable for a ticket or handoff doc.

**Task:** Using only `git log` and `git diff` variants, produce a complete picture of the repo: what the last 5 commits changed, current working tree state, and document findings as a short audit note.

**What I built:**

```bash
git log --oneline                  # get a full history overview
git diff HEAD~5 HEAD               # see net change across last 5 commits
git diff HEAD~4 HEAD               # narrow in on individual ranges
git diff HEAD~3 HEAD
git diff HEAD~2 HEAD
git diff HEAD~1 HEAD
git diff                           # check for unstaged changes
git diff --staged                  # check for staged changes
```

**What actually happened:**

`git diff HEAD HEAD~2` returned empty — initially looked like a bug. It's not. Those two commits are a change and its revert. The net diff is zero. The history shows the work happened, but the working tree state is identical to before. This is a real gotcha: diff alone can mislead you. The pattern is `git log` first to understand what happened, then `git diff` to inspect specific ranges.

Ran diffs at multiple `HEAD~N` offsets to isolate what each commit actually changed, since adjacent commits cancelled each other out.

**The result:**

```
Repo audit — git-lab — 2026-06-14

Last 5 commits: config experiments on lab2.txt.
- Two bad values added ("bad config line", "wrong config value") and reverted in sequence.
- One README update adding a session 2 header and description.
- Net change across all 5 commits: one line removed from lab2.txt ("wrong config value").

Working tree state:
- One staged change: "test edit" appended to lab2.txt (not yet committed).
- No unstaged changes.

Branch state: single branch (main), no remote configured.
Safe to work from — no in-progress merges, no untracked surprises.
```

**Why this approach and not another:** Ran diffs at multiple `HEAD~N` offsets rather than `git log -p` because I was building the mental model of relative navigation. In practice, `git log -p -5` would produce the same information in one command — message and diff together for each commit.

**What I'd do differently in production:** Use `git log -p -5` as the first pass — one command gives full commit messages and diffs together, faster than running multiple diff comparisons manually. Then use targeted `git show <hash>` to drill into any specific commit that needs closer inspection.

---

## Key Takeaways

- `git diff` compares working directory to staging area. After `git add`, it goes empty — that's correct. Use `git diff --staged` to see what's actually queued for the next commit.
- A zero diff between two commits doesn't mean nothing happened. A change and its revert cancel out to zero. Always read `git log` first, then use diff to inspect specific ranges.
- `HEAD~N` is relative navigation — faster than copying hashes for recent lookbacks. Use explicit hashes when comparing two specific non-adjacent points in history.
- `git log -p -5` is the audit command: last N commits with full diffs inline, one pass. Faster than running log then diff separately.
- `--decorate` surfaces ref labels. The useful signal is when `origin/main` and `main` appear on different commits — immediate visual confirmation of sync state without running `git status`.

---

## Where People Go Wrong

- **Running `git diff` after staging and assuming nothing changed:** The change is there — it moved to the staging area, where `git diff` can't see it. Always follow `git add` with `git diff --staged` before committing.
- **Trusting a zero diff without checking the log:** A net diff of zero between two commits can mean nothing changed, or it can mean a change and its revert cancel out. The log tells you which. The diff can't.
- **Using `git log` without `--all` on multi-branch repos:** Without `--all`, you only see the current branch. Other branches and remote tracking refs are invisible. `git log - oneline --graph --all --decorate` is the complete view.

---

## Senior Engineer Notes

- Before touching anything in an unfamiliar repo, run `git log --oneline -10`, then `git diff` and `git diff --staged`. Ten commits, current staged state, current unstaged state — full orientation in under 30 seconds.
- `git show <hash>` is the fastest way to inspect a single commit — message and diff in one output. Learn to reach for it before running a manual diff between two hashes.
- `git log -p --follow --<filename>` traces the full history of a single file, including across renames. That's the move when you need to know every change ever made to one specific file.

---

## Retain & Reinforce

- **Read:** [git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History) — the official reference for `git log` options. Skim the filtering section (`--author`, `--grep`, `- since`) — you'll reach for those in a real repo.
- **Explain:** Tell someone new to Git the difference between `git diff` and `git diff --staged` without using the words "staging area" — force yourself to use an analogy instead.
