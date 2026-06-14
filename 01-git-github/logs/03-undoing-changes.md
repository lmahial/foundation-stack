# Undoing Changes

**Skill area:** Git 
**Topic:** `git restore`, `git restore --staged`, `git reset` (soft/mixed/hard), `git revert`, `git stash` 
**Session:** 03

---

## The Big Picture

**In plain language:** Git gives you multiple tools to undo work — but each one operates on a different part of the three-tree architecture, and they have very different consequences. Some are safe and reversible. Some are permanent. Knowing which tool to reach for, and when, is what separates a calm recovery from a panicked mistake.

**Why it matters:** Bad commits happen. Staged changes get assembled wrong. Work needs to be shelved mid-task. In a team environment, using the wrong undo tool on shared history causes conflicts for everyone who's already pulled. The cost of picking the wrong tool isn't just personal — it breaks other people's workflows.

---

## The Concept

### Mental Model

Think of your work in three buckets — changes you haven't staged yet, changes you've staged but not committed, and changes you've committed. Each bucket has its own undo tool. The further right the bucket, the more powerful and potentially destructive the tool.

```
Working Directory  →  Staging Area  →  Repository (commits)
git restore           git restore       git reset
                      --staged          git revert
  
git stash  ←  shelves working directory + staging area temporarily
```

### How It Works

**`git restore <file>` — discard unstaged changes**

Throws away changes in the working directory. Reverts the file to whatever is in the staging area, or to the last commit if nothing is staged. No recovery. No reflog entry because the change was never committed.

```bash
git restore README.md
# Working directory change gone. Permanent. No undo.
```

**`git restore --staged <file>` — unstage a file**

Moves a file back out of the staging area. The change is preserved in the working directory — nothing is lost. This is always safe.

```bash
git restore --staged README.md
# File moves from staging area back to working directory
# Change is still there — just unstaged
```

**`git reset` — move HEAD backwards**

Moves HEAD (and the branch pointer) to a previous commit. Three modes with different levels of destruction:

```bash
git reset --soft HEAD~1
# HEAD moves back one commit
# Changes from that commit land in the staging area
# Working directory untouched
# Use when: you want to recommit with a cleaner message or combine with other changes

git reset --mixed HEAD~1
# HEAD moves back one commit (this is the default if no flag given)
# Staging area cleared — changes drop to working directory
# Working directory untouched
# Use when: you want to re-examine and re-stage selectively

git reset --hard HEAD~1
# HEAD moves back one commit
# Staging area wiped
# Working directory changes wiped
# Use when: the commit and all its changes are completely unwanted
# WARNING: changes are gone. Reflog can recover the commit, not the unstaged work.
```

**`git revert <hash>` — safe undo on shared history**

Creates a new commit that inverts the changes from a previous commit. The original commit stays in the log — both the mistake and the fix are recorded. History only moves forward. Safe on any branch others have pulled from.

```bash
git revert HEAD
# New commit created: "Revert <original message>"
# Original commit still in log
# Net effect on files: changes from that commit are undone
```

The structural difference from `reset`: `reset` removes commits from history. `revert` adds a commit to history. If you `reset` on a pushed branch, you rewrite history everyone else already has. `revert` never does that.

**`git stash` — shelve work in progress**

Saves the current working directory and staging area state to a stack, then cleans the tree. Use it when you need to switch context without committing half-finished work.

```bash
git stash push -m "description"   # named stash — always prefer this over plain git stash
git stash list                     # see all stashed contexts
git stash pop                      # restore the most recent stash and drop it from the stack
```

`stash pop` restores the work but drops the stash entry. If the pop fails due to a conflict, the stash entry is kept so nothing is lost.

**The deciding rule:**

|Situation|Tool|
|---|---|
|Unstaged change you don't want|`git restore <file>`|
|Staged change you want to unstage|`git restore --staged <file>`|
|Local commits not yet pushed|`git reset`|
|Commits already pushed to shared branch|`git revert`|
|Work in progress — need to switch context|`git stash push -m "description"`|

---

## Drills

### Drill 1 — `git restore`: discard unstaged changes

**What I did:**

```bash
echo "unwanted change" >> README.md
git status
git restore lab2.txt
git status
```

**Output:**

```
On branch main
Changes not staged for commit:
        modified:   lab2.txt
Untracked files:
        audit.txt

# after restore:
On branch main
Untracked files:
        audit.txt
nothing added to commit but untracked files present
```

**What this taught me:** `git restore` permanently removes the unstaged change. No recovery path — it never entered the commit graph so reflog can't help. The only safety check before running it is `git diff` to confirm exactly what you're about to lose.

---

### Drill 2 — `git restore --staged`: unstage without losing work

**What I did:**

```bash
echo "staged but not wanted" >> README.md
git add README.md
git status
git restore --staged README.md
git status
```

**Output:**

```
# after git add:
Changes to be committed:
        modified:   README.md

# after restore --staged:
Changes not staged for commit:
        modified:   README.md
```

**What this taught me:** `git restore --staged` only moves the file back out of the staging area. The change stays in the working directory — visible in `git diff`, not lost. This is always safe. The difference from plain `git restore`: one preserves the change, one destroys it.

---

### Drill 3 — `git reset --soft`: uncommit but keep staged

**What I did:**

```bash
echo "commit one" >> README.md && git add README.md && git commit -m "chore: temp commit one"
echo "commit two" >> README.md && git add README.md && git commit -m "chore: temp commit two"
git log --oneline -5
git reset --soft HEAD~2
git log --oneline -5
git status
```

**Output:**

```
# before reset:
0889093 (HEAD -> main) chore: temp commit two
94a3ddc chore: temp commit one
500e8a1 chore: add lab2 audit file
...

# after reset:
500e8a1 (HEAD -> main) chore: add lab2 audit file
...

# git status:
Changes to be committed:
        modified:   README.md
```

**What this taught me:** `--soft` unwraps the commits but leaves everything staged — like the commits were undone but the work is still packed and ready. Running `git commit` immediately after would create a single clean commit from both changes. This is the move when you've made several messy commits locally and want to squash them into one before pushing.

---

### Drill 4 — `git reset --mixed`: uncommit and unstage

**What I did:**

```bash
git commit -m "chore: recombined temp commits"
echo "mixed test" >> README.md && git add README.md && git commit -m "chore: mixed reset target"
git log --oneline -4
git reset --mixed HEAD~1
git log --oneline -4
git status
```

**Output:**

```
# before reset:
aa0e60d (HEAD -> main) chore: mixed reset target
7714e25 chore: recombined temp commits
...

# after reset:
Unstaged changes after reset:
M       README.md

7714e25 (HEAD -> main) chore: recombined temp commits
...

# git status:
Changes not staged for commit:
        modified:   README.md
```

**What this taught me:** `--mixed` moves HEAD back and drops the changes to the working directory — unstaged but not lost. One step more destructive than `--soft`: the changes are no longer staged, so you have to re-examine and re-add deliberately. Use it when you want to split one commit into multiple focused commits.

---

### Drill 5 — `git reset --hard`: uncommit and wipe

**What I did:**

```bash
git restore README.md
echo "hard reset target" >> README.md && git add README.md && git commit -m "chore: hard reset target"
git log --oneline -4
git reset --hard HEAD~1
git log --oneline -4
git status
cat README.md
```

**Output:**

```
# before reset:
f9c61d8 (HEAD -> main) chore: hard reset target
7714e25 chore: recombined temp commits
...

HEAD is now at 7714e25 chore: recombined temp commits

# after reset:
7714e25 (HEAD -> main) chore: recombined temp commits
...

# git status:
nothing to commit, working tree clean

# cat README.md — "hard reset target" line is gone
```

**What this taught me:** `--hard` wipes the commit, the staging area, and the working directory change in one move. Clean status, nothing in diff, nothing in `git diff --staged`. The commit can be recovered via reflog (the hash still exists for ~90 days), but any unstaged working directory changes that weren't committed are unrecoverable. Never use `--hard` on a pushed branch.

---

### Drill 6 — `git revert`: safe undo on shared history

**What I did:**

```bash
git log --oneline -4
git revert HEAD
git log --oneline -4
cat README.md
```

**Output:**

```
# before revert:
7714e25 (HEAD -> main) chore: recombined temp commits
500e8a1 chore: add lab2 audit file
...

# after revert:
532d589 (HEAD -> main) Revert "chore: recombined temp commits"
7714e25 chore: recombined temp commits
500e8a1 chore: add lab2 audit file
...
```

**What this taught me:** `revert` adds a new commit — the original stays in the log. Both the mistake and the fix are recorded. `reset` removes commits from history; `revert` only adds to it. That's the property that makes it safe on shared branches. If you `reset` on a pushed branch, you rewrite history everyone else already has locally — their next push will conflict. `revert` never causes that.

---

### Drill 7 — `git stash`: shelve work in progress

**What I did:**

```bash
echo "work in progress" >> README.md
git status
git stash
git status
git stash pop
git status
cat README.md
```

**Output:**

```
# before stash:
Changes not staged for commit:
        modified:   README.md

# after stash:
Saved working directory and index state WIP on main: 532d589 ...
nothing to commit, working tree clean

# after stash pop:
Changes not staged for commit:
        modified:   README.md
# "work in progress" line restored in README.md
```

**What this taught me:** Stash cleans the working tree without committing — the change goes onto a stack and the tree looks clean to Git. `stash pop` restores it exactly. The problem stash solves: you're mid-feature and need to switch branches or investigate a bug without committing half-finished work. Committing just to switch context pollutes the history with WIP commits that then need to be cleaned up.

---

## Lab

**Scenario:** Working on a config file update mid-way through. Staged something that shouldn't be staged, made a bad commit, then got pulled away to investigate something urgent. Need to clean up without losing real work and without leaving the repo broken.

**What I built:**

```bash
# Step 1 — two changes, first staged, second unstaged
echo "lab in progress" >> README.md
git add README.md
echo "lab reset-revert-stash work" >> README.md
git status
# Result: README.md appears in both "Changes to be committed" and "Changes not staged"

# Step 2 — unstage the staged change
git restore --staged README.md
git status
# Result: both changes now unstaged together in working directory

# Step 3 — commit both changes cleanly
git add README.md
git commit -m "lab: undoing changes"

# Step 4 — make a commit in error, then soft reset
echo "commit in error" >> README.md
git add README.md
git commit -m "lab: commit mistake"
git reset --soft HEAD~1
git status
# Result: "commit in error" change back in staging area, commit gone from log

# Step 5 — stash the staged work, confirm clean tree, restore
git stash push -m "lab work"
git status
# Result: nothing to commit, working tree clean
git stash pop
git status
# Result: README.md modified, change restored
```

**What actually happened:**

Step 2 had a scope issue — `git restore --staged README.md` unstaged everything together rather than isolating just the first change. Both lines ended up in the working directory. Then both got committed together in step 3. The commands were correct; the granularity was off. The right approach when a file has multiple hunks and only some should be staged: `git add -p` — interactive patch staging that lets you select individual hunks. Full topic for another session, but the gap is noted.

Everything else executed cleanly. `--soft` kept the change staged as expected. Named stash with `-m "lab work"` was the right call — readable in `git stash list` if context switching takes a while.

**The result:**

```
e6c379c (HEAD -> main) lab: undoing changes
532d589 Revert "chore: recombined temp commits"
7714e25 chore: recombined temp commits
500e8a1 chore: add lab2 audit file
57dbcc3 Revert "chore: add wrong config — revert candidate"
747e3c5 chore: add wrong config — revert candidate

# git status:
Changes not staged for commit:
        modified:   README.md
```

**Why this approach and not another:** `--soft` over `--hard` for the bad commit because the change had real content that needed to be recommitted cleanly. `--hard` would have wiped it. Named stash over anonymous because any stash you can't read in `git stash list` is a liability the moment you have two of them.

**What I'd do differently in production:** Use `git add -p` from the start when a file has multiple unrelated changes that need to go into separate commits. Staging whole files when hunks need to be split is a habit that creates messy history.

---

## Key Takeaways

- `git restore` is the only common Git operation with no recovery path — the change was never committed, so reflog can't help. Run `git diff` first to confirm exactly what you're about to lose.
- `git restore --staged` is always safe — it only moves the file out of staging, the change stays in the working directory.
- `reset` modes map to buckets: `--soft` stops at staging, `--mixed` stops at working directory, `--hard` wipes everything. The mode determines how far back the destruction goes.
- `revert` is the only correct undo on pushed/shared history. It extends the log forward. `reset` rewrites history — on a shared branch, that breaks everyone who's already pulled.
- `git stash push -m "description"` over plain `git stash` any time you might have more than one stashed context. Anonymous stashes become unreadable fast.

---

## Where People Go Wrong

- **`git reset --hard` on a pushed branch:** You rewrite history your teammates already have locally. Their next push conflicts with the remote. If it's pushed, `revert` is the only safe move — no exceptions.
- **Running `git restore` without checking `git diff` first:** No confirmation prompt, no undo. The file reverts instantly. Always know what you're about to destroy before you destroy it.
- **Plain `git stash` with multiple contexts:** `git stash list` shows `stash@{0}`, `stash@{1}`, `stash@{2}` with no descriptions. You're guessing which is which. Named stashes cost two extra words and save real confusion.

---

## Senior Engineer Notes

- Before any `reset`, run `git log --oneline -5` and `git status` first. Confirm exactly what HEAD is and what state the tree is in before moving it. Five seconds of checking prevents targeting the wrong commit.
- `git add -p` is the granular staging tool — it opens an interactive hunk selector so you can stage part of a file's changes and leave the rest unstaged. Essential when one file has two unrelated changes that belong in separate commits.
- If you `reset --hard` and immediately regret it, run `git reflog` — the commit hash is still there for ~90 days. You can `git reset --hard <hash>` back to it. The window exists; don't rely on it, but know it's there.

---

## Retain-Reinforce

Read: [git-scm.com/book/en/v2/Git-Tools-Reset-Demystified](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified) — the canonical explanation of how reset interacts with the three-tree architecture. Read the diagrams.
