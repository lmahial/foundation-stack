# Branching — `git branch`, `git switch`, `HEAD`

**Skill area:** Git
**Topic:** `git branch`, `git switch`, `git switch -c`, `git branch -d/-D`, `git branch -v`, `git branch --merged`
**Session:** 04



## The Big Picture

**In plain language:** A branch in Git is not a copy of the codebase. It's a lightweight pointer — a named reference to a single commit. Creating a branch costs nothing because nothing is duplicated. Git just writes a new pointer. Understanding this changes how you reason about branching strategy and why Git branching is fast where other version control systems are slow.

**Why it matters:** Every team workflow — feature branches, hotfix branches, release branches — depends on this model. If you think branches are copies, you'll treat them as expensive and avoid them. If you know they're pointers, you'll use them freely and correctly. The wrong mental model leads to working directly on `main`, which is how production breaks.



## The Concept

### Mental Model

A branch is a sticky note attached to a commit. Every time you make a new commit, Git moves that sticky note forward to the new commit automatically. HEAD is a second sticky note that points to whichever branch you're currently on — it moves with you when you switch branches.

When you create a branch, Git writes a new sticky note pointing to the same commit you're on right now. No files copied. No directories duplicated. Two sticky notes, same commit. That's why creating a branch is instantaneous.

```
Before branch:          After git branch feature:    After commit on feature:
                                                      
A - B - C  (main)       A - B - C  (main)            A - B - C  (main)
            ↑                       ↑                             ↑
           HEAD                  feature                       feature - D
                                    ↑                                      ↑
                                  HEAD                                   HEAD
```

### How It Works

**What HEAD is**

HEAD is a pointer to the branch you're currently on. When you switch branches, HEAD moves to point at the new branch. When you commit, the branch HEAD is pointing to moves forward to the new commit. HEAD follows it.

Detached HEAD is when HEAD points directly to a commit instead of a branch. This happens when you checkout a specific hash or tag. Commits made in detached HEAD state aren't on any branch — switch away without creating a branch first and those commits become unreachable.

**Listing and creating branches**

```bash
git branch              # list all local branches — * marks current
git branch -v           # list with last commit hash and message per branch
git branch --merged     # list branches fully contained in current branch's history
git branch <name>       # create a new branch at current commit — does NOT switch
```

**Switching branches**

```bash
git switch <name>       # move HEAD to that branch, update working directory
git switch -c <name>    # create and switch in one step — the one you actually use
```

When you switch branches, Git updates your working directory to match the state of the commit the target branch points to. Files appear and disappear based on what exists in that branch's history.

**Deleting branches**

```bash
git branch -d <name>    # safe delete — refuses if branch has unmerged commits
git branch -D <name>    # force delete — no safety check, unmerged commits become unreachable
```

`-d` checks whether all commits on the branch are reachable from the current branch. If they are, the branch is safe to delete — the commits still exist, just the pointer is removed. If there are unmerged commits, `-d` exits with an error. `-D` skips that check entirely.

**`git branch --merged` — the cleanup tool**

Lists branches whose commits are fully contained in the current branch's history. After merging a feature branch, it appears in `--merged` — that's the signal it's safe to delete. Always run this before bulk branch cleanup.

Note: `--merged` is relative to the branch you're on. From `feature/session-4-lab`, `main` shows as merged because `main` is an ancestor — all of main's commits are contained within the feature branch's history. Switch to `main` and re-run to get the view that matters for cleanup.



## Drills

### Drill 1 — `git branch`: create without switching

**What I did:**

```bash
git branch
git log --oneline -3
git branch feature/drill1
git branch
git log --oneline -3
```

**Output:**

```
# before:
* main

e6c379c (HEAD -> main) lab: undoing changes
532d589 Revert "chore: recombined temp commits"
7714e25 chore: recombined temp commits

# after git branch feature/drill1:
  feature/drill1
* main

e6c379c (HEAD -> main, feature/drill1) lab: undoing changes
532d589 Revert "chore: recombined temp commits"
```

**What this taught me:** Creating a branch doesn't switch to it — HEAD stayed on `main`. Both `main` and `feature/drill1` now point at the same commit `e6c379c`. No files were copied. The log shows both labels on the same line, confirming they're two pointers to one commit.



### Drill 2 — `git switch`: move HEAD

**What I did:**

```bash
git switch feature/drill1
git branch
git log --oneline -3
```

**Output:**

```
Switched to branch 'feature/drill1'

* feature/drill1
  main

e6c379c (HEAD -> feature/drill1, main) lab: undoing changes
532d589 Revert "chore: recombined temp commits"
7714e25 chore: recombined temp commits
```

**What this taught me:** `git switch` moved HEAD to `feature/drill1`. `main` didn't move — it has no reason to. The commit is the same. Only the HEAD pointer changed. The log shows `HEAD -> feature/drill1` with `main` alongside it on the same commit.



### Drill 3 — commit on a branch: pointers diverge

**What I did:**

```bash
echo "feature work" >> README.md
git add README.md
git commit -m "feat: add feature work on drill1 branch"
git log --oneline --graph --all --decorate
```

**Output:**

```
* 51b878b (HEAD -> feature/drill1) feat: add feature work on drill1 branch
* e6c379c (main) lab: undoing changes
* 532d589 Revert "chore: recombined temp commits"
```

**What this taught me:** After committing on `feature/drill1`, the two pointers diverged. `feature/drill1` moved forward to `51b878b`. `main` stayed on `e6c379c`. HEAD followed `feature/drill1`. This is the core mechanic — branches diverge from a shared ancestor as soon as independent commits land on each.



### Drill 4 — `git switch` back: working directory updates

**What I did:**

```bash
git switch main
cat README.md
git log --oneline --graph --all --decorate
```

**Output:**

```
Switched to branch 'main'

# README.md — "feature work" line is gone

* 51b878b (feature/drill1) feat: add feature work on drill1 branch
* e6c379c (HEAD -> main) lab: undoing changes
* 532d589 Revert "chore: recombined temp commits"
```

**What this taught me:** Switching branches updates the working directory to match the target branch's commit state. The "feature work" line disappeared from README because `main` doesn't have that commit. The file isn't gone — it's on `feature/drill1` exactly where it was committed. Git swapped the working directory contents on switch.



### Drill 5 — `git branch -d` vs `-D`

**What I did:**

```bash
# safe delete on unmerged branch — fails
git branch -d feature/drill1

# create throwaway branch with unmerged commit
git switch -c throwaway
echo "unmerged work" >> README.md
git add README.md
git commit -m "chore: unmerged throwaway commit"
git switch main

# safe delete — fails
git branch -d throwaway

# force delete
git branch -D throwaway

git branch
```

**Output:**

```
error: the branch 'feature/drill1' is not fully merged
hint: If you are sure you want to delete it, run 'git branch -D feature/drill1'

Switched to a new branch 'throwaway'
[throwaway 4da3f86] chore: unmerged throwaway commit

error: the branch 'throwaway' is not fully merged
hint: If you are sure you want to delete it, run 'git branch -D throwaway'

Deleted branch throwaway (was 4da3f86).

  feature/drill1
* main
```

**What this taught me:** `-d` checks whether all commits on the branch are reachable from the current branch before deleting. If they're not — meaning there's work that exists nowhere else — it exits with an error. `-D` skips that check entirely. The risk: any commits that exist only on the force-deleted branch become unreachable. *Reflog* can recover them for ~90 days, but after that they're gone.



### Drill 6 — `git branch -v` and `--merged`

**What I did:**

```bash
git branch -v
git branch --merged
```

**Output:**

```
  feature/drill1 51b878b feat: add feature work on drill1 branch
* main           e6c379c lab: undoing changes

* main
```

**What this taught me:** `git branch -v` gives a quick snapshot of every branch and where it's pointing — useful for orientation in a repo with many branches. `--merged` shows only branches fully contained in the current branch's history. `feature/drill1` isn't listed because it has `51b878b`, which `main` doesn't have. The cleanup workflow is: merge, run `--merged`, delete everything listed except `main`.



## Lab

**Scenario:** Starting work on a new feature. Team convention: all work happens on a named branch, nothing commits directly to `main`. Set up the branch, do the work, verify state, prepare for handoff without merging.

**What I built:**

```bash
# create and switch in one command
git switch -c feature/session-4-lab

# two commits on the feature branch
echo "test file for lab" >> lab.txt
git add lab.txt
git commit -m "chore(lab): add lab.txt - branch session"

echo "this is a test branch for session 4 lab - merging" >> README.md
git add README.md
git commit -m "chore(lab): update readme with little description for lab part"

# verify divergence
git log --oneline --graph --all --decorate

# check merged status from feature branch
git branch --merged

# switch back to main, confirm working directory state
git switch main
cat README.md
git branch --merged
git branch -v
```

**What actually happened:**

First commit attempt on README failed — ran `git add readme.md` (lowercase) but the file is `README.md`. Git on macOS didn't error on the add but the commit found nothing staged because the case didn't match the tracked filename. Caught it, corrected to `README.md`, committed cleanly.

`git branch --merged` from `feature/session-4-lab` showed `main` as merged — initially unexpected. It's correct: `main` is an ancestor of the feature branch, so all of main's commits are contained within it. `--merged` is relative to where you are. Switched to `main`and re-ran — only `main` appeared, confirming neither feature branch has been merged into `main` yet.

**The result:**

```
* 44785c1 (feature/session-4-lab) chore(lab): update readme with little description for lab part
* 4b3697f chore(lab): add lab.txt - branch session
| * 51b878b (feature/drill1) feat: add feature work on drill1 branch
|/
* e6c379c (HEAD -> main) lab: undoing changes

# git branch -v:
  feature/drill1        51b878b feat: add feature work on drill1 branch
  feature/session-4-lab 44785c1 chore(lab): update readme with little description for lab part
* main                  e6c379c lab: undoing changes

# cat README.md on main — feature branch changes not present
```

**Why this approach and not another:** `git switch -c` over two separate commands — one command, one mental step, no risk of forgetting to switch after creating. The branch naming follows the team convention from the project docs: `feature/<description>`.

**What I'd do differently in production:** Branch names would follow the team's ticket convention — `feature/PROJ-123-description` rather than a free-form name. Makes it trivial to trace a branch back to its ticket without reading the commit history.



## Key Takeaways

- A branch is a pointer to a commit, not a copy of the codebase. Creating one is instantaneous — Git writes a reference, nothing more.
- HEAD points to the current branch. Switching branches moves HEAD and updates the working directory to match the target commit's state. Files appear and disappear based on branch history.
- `git switch -c <name>` is the command you actually use — create and switch in one step.
- `-d` is safe: refuses to delete a branch with unmerged commits. `-D` is force: no check, unmerged commits become unreachable. Default to `-d` and only reach for `-D` when you're certain.
- `git branch --merged` is relative to the branch you're on. Run it from `main` before cleanup — that's the view that tells you what's safe to delete.



## Where People Go Wrong

- **Committing directly to `main`:** Without a branch, there's no isolation. A bad commit goes straight into the shared history. The habit to build now: `git switch -c feature/<name>` before any new work, every time.
- **Using `-D` without checking `-d` first:** If `-d` refuses, read the error before reaching for `-D`. The error is telling you there are commits that exist nowhere else. Know what you're deleting before you delete it.
- **Misreading `--merged` from the wrong branch:** `--merged` from a feature branch shows `main` as merged because main is an ancestor. Always run `--merged` from `main` when deciding what to clean up.



## Senior Engineer Notes

- In a team repo, `git branch -v` and `git branch --merged` before any cleanup session. Never force-delete without knowing what's on the branch. The thirty seconds of checking is cheaper than recovering lost commits from reflog.
- macOS has a case-insensitive filesystem — `git add readme.md` succeeds even if the tracked file is `README.md`. On a Linux server it fails. Always match case exactly when staging files, or use `git add .` for the whole directory when appropriate.
- The branch naming convention is a team contract. Whatever it is — `feature/`, `fix/`, `PROJ-123-` — follow it consistently. Branch names show up in PR titles, CI pipelines, and deployment logs. A readable branch name saves real time during incident response.


## Retain & Reinforce

Read: [git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell) — the official explanation with diagrams of how the pointer model works. The diagrams are worth the read.