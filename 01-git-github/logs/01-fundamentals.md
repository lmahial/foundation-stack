# Git & GitHub — Fundamentals

**Skill area:** Git  
**Topic:** Snapshots, three-tree architecture, staging, commits, .gitignore  
**Session:** 01

---

## The Big Picture

**In plain language:** Git is a version control system that tracks the history of your project as a series of snapshots. Every time you commit, Git saves a complete picture of your project at that moment. If a file didn't change, Git points to the previous snapshot rather than storing it again — so history is efficient without losing anything.

**Why it matters:** Every infrastructure repo — configs, scripts, Terraform, Ansible — lives in Git. Without it, you have no history, no way to recover from a bad change, no way to collaborate, and no audit trail. In production, "what changed and when" is the first question after any incident. Git is how you answer it.

---

## The Concept

### Mental Model

Git is not a changelog — it's a timeline of photographs. Each commit is a snapshot of the entire project at that moment in time. You can walk backward through those photos to see exactly what existed at any point in history.

The three-tree architecture is how Git organizes work at any given moment:

- **Working directory** — your files on disk. Where you make changes.
- **Staging area (index)** — a deliberate holding area. You choose exactly what goes into the next snapshot.
- **Repository (.git)** — the permanent record. Every committed snapshot lives here.

The flow is always: edit → stage → commit. Every Git command either moves something between these trees or reads from them.

```bash
# Initialize a new repo — creates the .git directory
git init

# Check which tree each file is in right now
git status

# Move a file from working directory to staging
git add README.md

# Take the snapshot — move staged changes into the repo
git commit -m "docs(readme): initial commit — add README"

# Read the commit history
git log --oneline
```

**HEAD** is a pointer to your current branch. It answers "where am I in the history" — not "is anything pending." On a brand new repo, HEAD points to a branch that doesn't exist yet. The branch is created the moment the first commit is made.

**`.gitignore`** defines what Git will never track. The key rule: if a file contains credentials or is auto-generated, it belongs in `.gitignore` before the first commit — not after. Once a file is committed, it's in history forever. Deleting it later doesn't remove it. Removing credentials from Git history requires rewriting history and rotating the credentials.

Standard entries for an infrastructure repo:

```
.env                # credentials and environment variables
*.pem               # private keys
*.key               # private keys
logs/               # generated log output
*.pyc               # Python bytecode
__pycache__/        # Python cache directories
.venv/              # Python virtual environments
*.tfstate           # Terraform state — contains resource IDs, sometimes secrets
*.tfstate.backup    # Terraform state backups
.terraform/         # Terraform provider cache
.DS_Store           # macOS metadata noise
```

**`.gitkeep`** is an empty placeholder file used to track empty directories in Git. Git tracks files, not directories — so an empty directory won't appear in the repo without a file inside it. The name `.gitkeep` is a community convention, not a Git feature.

```bash
mkdir session-logs
touch session-logs/.gitkeep
git add session-logs/.gitkeep
```

---

## Drills

### Drill 1 — Observe

**What I did:**

```bash
mkdir ~/git-lab && cd ~/git-lab
git init
ls -la
cat .git/HEAD
```

**Output:**

```
ref: refs/heads/main
```

**What this taught me:** HEAD points to the current branch — in this case main. On a fresh repo with no commits, main doesn't exist yet as a real branch. HEAD is pointing to a reference that hasn't been created. The branch becomes real the moment the first commit is made.

---

### Drill 2 — Stage and Commit

**What I did:**

```bash
echo "# My Lab Repo" > README.md
git status
git add README.md
git status
git commit -m "docs(readme): initial commit — add README"
git log
```

**Output (first git status):**

```
On branch main
No commits yet
Untracked files:
        README.md
nothing added to commit but untracked files present
```

**Output (second git status):**

```
On branch main
No commits yet
Changes to be committed:
        new file: README.md
```

**What this taught me:** The first status shows README.md is in the working directory but Git isn't tracking it. After `git add`, it moves to the staging area — Git now knows it exists and has it queued for the next commit. The shell prompt indicators showed this too: `?1` for untracked, `+1` for staged.

---

### Drill 3 — Break It

**What I did:**

```bash
mkdir logs
echo "error: disk full" > logs/system.log
echo "DB_PASSWORD=<redacted>" > .env
git status
```

Then created `.gitignore`:

```
.env
logs/
*.log
.DS_Store
```

```bash
git add .gitignore
git commit -m "chore: add .gitignore"
git status
```

**Output (after commit):**

```
On branch main
nothing to commit, working tree clean
```

**What this taught me:** `.env` and the logs directory are now permanently excluded from tracking. The reason `.env`specifically belongs in `.gitignore` and not just left unstaged: if it's ever staged and committed even once, the credential is in the permanent history. Removing the file doesn't fix it. `.gitignore` prevents the accident before it can happen.

---

### Drill 4 — Modify

**What I did:**

```bash
vim README.md   # added two lines describing the repo
git add README.md
git commit -m "chore(readme): add description to readme"
git log --oneline
```

**Output:**

```
1868ff4 (HEAD -> main) chore(readme): add description to readme
41e590f chore: add .gitignore
effdd24 docs(readme): initial commit — add README
```

**What this taught me:** Commit messages exist for the person reading the log six months from now — including yourself. `"update README"` tells you nothing without opening the diff. `"chore(readme): add description to readme"` tells you the type of change, the scope, and what happened. The log becomes readable history rather than noise.

---

### Drill 5 — Extend

**My idea:** Add `.gitignore` entries for file types that realistically appear in infrastructure repos and should never be committed.

**What I did:**

```bash
vim .gitignore
# added: .venv/, *.pyc, __pycache__/, *.tfstate, *.pem
git add .gitignore
git commit -m "chore(gitignore): add venv and macOS folder attributes to ignore"
git log --oneline
```

**What this taught me:** `.venv/` is auto-generated and can be hundreds of MB. `.DS_Store` is macOS metadata with no value to anyone else. Both are the category of file that causes noise and risk if committed — generated automatically, never meaningful to the repo's purpose.

---

## Lab

**Scenario:** Setting up `foundation-stack` — the repo that holds all Block 1–5 work. Not a throwaway practice repo. The real one.

**Task:** Repo with correct directory structure, appropriate README, solid `.gitignore`, minimum three commits with proper messages.

**What I built:**

```bash
mkdir foundations-stack && cd foundations-stack
git init

vim README.md
git add README.md
git commit -m "docs(readme): initial commit — add README"

vim .gitignore
git add .gitignore
git commit -m "chore(gitignore): add files to be ignored from tracking"

mkdir session-logs && touch session-logs/.gitkeep
mkdir scripts && touch scripts/.gitkeep
git add scripts/.gitkeep session-logs/.gitkeep
git commit -m "chore: add session-logs and scripts directories to build repo structure"

vim .gitignore  # expanded entries
git add .gitignore
git commit -m "chore(gitignore): add additional files to be removed from tracking"
```

**What actually happened:** Attempted to pass the commit message into `git add` on the first `.gitignore` commit — `git add .gitignore "chore(...)"` — Git returned a pathspec error. Caught it, ran `git add .gitignore` separately, then committed correctly.

**The result:**

```
2913f18 (HEAD -> main) chore(gitignore): add additional files to be removed from tracking
fc223ed chore: add session-logs and scripts directories to build repo structure
c51e018 chore(gitignore): add files to be ignored from tracking
b1c0af2 docs(readme): initial commit — add README
```

**Why this approach and not another:** Built README and `.gitignore` first before any structure — those define what the repo is and what it will never track before any real content arrives. Directory scaffold came after, using `.gitkeep` to hold empty directories in Git.

**What I'd do differently in production:** A shared team repo would have branch protection on main from day one. Nothing goes directly to main — all changes via pull request, even for solo work. Builds the habit before it's required.

---

## Where People Go Wrong

- **Staging credentials by accident:** `.env` committed once means the secret is in history permanently. The fix is history rewriting and credential rotation — not a one-minute job. `.gitignore` before the first commit, every time.
- **Ignoring specific filenames instead of patterns:** Writing `logs/system.log` instead of `logs/` means the next log file that lands there is untracked again. Ignore directories and extensions, not individual filenames.
- **Treating commit messages as notes to yourself right now:** Messages like `"update README"` or `"fix stuff"` are useless in a log with 200 entries. Write for the engineer doing the post-incident review — or yourself six months from now at 11pm trying to figure out what changed.

---

## Key Takeaways

- Git tracks snapshots, not diffs. Each commit is a complete picture of the project at that moment.
- The three-tree model — working directory, staging, repository — is the mental model behind every Git command. Always know which tree you're working in.
- `HEAD` answers "where am I" — not "is anything pending." It points to the current branch, nothing more.
- `.gitignore` prevents accidents before they happen. Credentials in history are a crisis, not a mistake you can quietly undo.

---

## Senior Engineer Notes

- The staging area feels like friction at first. It isn't. It's what lets you commit `nginx.conf` without accidentally committing the `.env` sitting next to it. Use it deliberately — stage exactly what belongs in this snapshot, nothing more.
- `git status` before every commit. Experienced engineers don't skip this. It takes two seconds and catches the class of mistake that takes twenty minutes to undo.
- Commit messages are written for the post-incident review, not for right now. `type(scope): what happened` in one line. Future you will either thank you or curse you — the message is the difference.

---

## Retain & Reinforce

- **Explain it:** Without looking at your notes, describe the three-tree architecture in two sentences as if you're onboarding a junior teammate. Write it down or say it out loud. If you stumble, that's what to review.
- **Read:** [Git - Recording Changes to the Repository](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository) — the official Pro Git book, chapter on staging and committing. Different angle than what we covered, same concepts.

---
