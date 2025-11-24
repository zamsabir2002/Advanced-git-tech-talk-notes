
> 🧭 Run everything in **PowerShell**. Commands that manipulate the demo folder use PowerShell-safe patterns.
> If you want a single script file later, I can produce that — for now this is organized as slide-by-slide / demo-by-demo steps you can copy/paste.

---

# Workshop setup (do this once at start)

```powershell
# CLEAN START (PowerShell-safe)
if (Test-Path "advanced-git-demo") { Remove-Item -Recurse -Force "advanced-git-demo" }
md advanced-git-demo
cd advanced-git-demo
git init
git config user.name "Demo User"
git config user.email "demo@example.com"
```

Presenter note: “We start with a fresh repo so each demo is reproducible. Ask audience to open PowerShell and follow along.”

---

# SECTION 0 — Git Internals (practical / visual)

Goal: make the snapshot model tangible (blobs, trees, commit objects, index).

## Demo 0.1 — Create baseline & inspect objects

```powershell
# Create files and initial commit
"Hello Git Internals" | Out-File file.txt -Encoding utf8
"console.log('v1');" | Out-File app.js -Encoding utf8
md src
"function lib(){ return 1 }" | Out-File src/lib.js -Encoding utf8

git add .
git ls-files --stage

// output: 100644 blob 5a1bc98fd3e1f6876ad1a8f9...  file1.txt



git commit -m "chore: initial commit"
```

Show:

```powershell
# Show commit object
git cat-file -p HEAD

# Show root tree of HEAD
git cat-file -p (git rev-parse HEAD^{tree})

# Show tree listing (human-friendly)
git ls-tree -r HEAD

# Show a blob content (pick a blob hash from ls-tree)
# e.g., replace <blob-sha> with one returned above
git cat-file -p <blob-sha>
```

What to show/explain:

* `git cat-file -p HEAD` prints commit metadata (author, committer, tree SHA, parent if any).
* `git ls-tree -r HEAD` lists blobs & trees with SHA—point out the “blob” line has only content hash, mode, and filename (no filename is stored inside blob itself).
* `git hash-object file.txt` produces the blob SHA — show it matches a file under `.git/objects/<first2>/<remaining>`.

Presenter notes:

* Short verbal line: “Git stores snapshots as objects. A commit points to a tree; trees point to blobs (file contents).”
* Why this matters: explains deduplication and why branches are cheap.
* Audience exercise (5 min): run `git ls-tree -r HEAD` and `git cat-file -p <tree-sha>` to trace one file from commit -> tree -> blob.

Troubleshooting:

* If `git cat-file` complains about SHA, re-run `git ls-tree -r HEAD` and copy a valid SHA.

---

## Demo 0.2 — Index / staging area visualization

```powershell
# Make a change to app.js but don't add yet
Add-Content app.js "`n// added line for staging demo"

# Show status and index
git status --short
git ls-files -s

# Stage the change
git add app.js

# Show index after staging
git ls-files -s

# Commit
git commit -m "feat: update app.js (staging demo)"
git log --oneline -n 3
```

What to show/explain:

* `git status --short` shows working tree status: ` M app.js` before staging.
* `git ls-files -s` shows index entries: staged blobs with their SHA.
* Emphasize: `git add` writes blob and updates index; HEAD unchanged until commit.

Presenter notes:

* Short: “Index is the snapshot-to-be. Useful to craft commits precisely.”
* Audience exercise: modify a file and stage only part of it using `git add -p` (we’ll do that later in Productivity).

---

# SECTION 1 — Git Power Essentials (Grouped Basics)

> For each subtopic below: run the commands, show outputs, then explain what changed visually (status, log, objects).

---

## Group 1 — Quick Workflow Shortcuts

### Demo 1.1 — `git stash` / `git stash pop` (detailed)

```powershell
# Starting point (ensure clean)
git checkout -b demo-stash
git reset --hard HEAD

# Make staged & unstaged changes
Add-Content app.js "`n// WIP: change 1 (unstaged)"
Add-Content src/lib.js "`n// WIP: change 2 (unstaged)"
git add src/lib.js                 # stage only lib.js

# Show before stash
git status --short
git diff --staged

# Create a named stash including untracked
git stash push -m "WIP: demo-stash changes" -u

# Show stash list
git stash list
git stash show -p stash@{0}

# Work on another branch while stash exists
git checkout main
# Simulate quick hotfix
Add-Content file.txt "`nhotfix"
git add file.txt
git commit -m "chore: quick hotfix"

# Return and apply stash
git checkout demo-stash
git stash pop                        # apply and remove the most recent stash
git status --short
git restore --staged src/lib.js      # optional: unstage if needed
```

Key extra commands to explain:

```powershell
# Apply without dropping
git stash apply stash@{0}

# Drop a stash if you no longer need it
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

Presenter notes:

* Explain stash stores two commits: index state and working tree state (plus possible untracked commit when -u used).
* Why matters: safe, temporary storage while context switching.
* Audience exercise (5–7 min): create a stash with untracked files, switch branch, and demonstrate `git stash pop` conflicts if both branches touched same file. Show how to resolve.

Troubleshooting:

* If `git stash pop` yields conflicts: resolve as normal merge conflicts, then `git add` and `git commit` if needed.

---

### Demo 1.2 — `git checkout -` (toggle previous branch)

```powershell
# Create two branches and toggle
git checkout -b feature/A
git checkout main
Write-Output "on main line" | Out-File main-demo.txt -Append
git add main-demo.txt
git commit -m "chore: touched main"

# Toggle back to previous branch
git checkout -
# Then toggle again
git checkout -
```

Presenter notes:

* `git checkout -` is shorthand for `git checkout @{-1}`, which is handy during code review or when toggling between a PR and a local branch.
* Audience exercise (2 min): toggle between two branches and show `git branch --show-current`.

---

### Demo 1.3 — `git commit --amend` and variants

```powershell
# Make a small change, stage
Add-Content app.js "`n// amend demo line"
git add app.js

# Amend and keep message
git commit --amend --no-edit

# Amend and change message
git commit --amend -m "feat: amended commit with additional change"

# Show that HEAD changed (new SHA)
git log --oneline -n 3
```

Explain:

* `--amend` creates new commit replacing the previous HEAD commit. If branch already pushed, you must `force` push, which is risky.

Presenter notes:

* Why matters: cleanup small mistakes before pushing. Warn strongly about amending public commits.
* Audience exercise: create a commit, push to a throwaway remote or simulate pushing by just noting hashes, then amend and show new hash.

---

## Group 2 — Productivity Boosters

### Demo 2.1 — Git aliases (practical)

```powershell
# Create helpful aliases
git config --global alias.ac 'commit -am'
git config --global alias.lg 'log --graph --oneline --decorate --all'
git config --global alias.st 'status -sb'

# Use them
git ac "chore: demo alias ac"
git lg -n 5
git st
```

Presenter notes:

* Encourage consistent team aliases in shared dotfiles; show how to view `~/.gitconfig`.
* Exercise: ask audience to set `git config --global alias.co checkout` then use `git co -b demo-co`.

---

### Demo 2.2 — `git add -p` (patch mode, deep)

```powershell
# Prepare a file with multiple changes
Add-Content file.txt "`nline A"
Add-Content file.txt "`nline B"
Add-Content file.txt "`nline C"

# Run interactive staging
git add -p

# Walk through options: y, n, s, e, q
# After staging one hunk:
git status --short
git diff --staged
git commit -m "feat: staged selective hunks"
```

Explain: splitting hunks allows atomic commits even when changes are in same file.

Presenter notes:

* Live demo: press `s` to split a hunk then `y` for one and `n` for another.
* Exercise: give 3 small independent changes in one file and ask volunteers to stage only the bugfix hunk and commit.

---

## Group 3 — Safe Undo & Log Exploration

### Demo 3.1 — `git revert` vs `git reset` (in depth)

```powershell
# Create two commits to operate on
Add-Content file.txt "`nrevert-demo line"
git add file.txt
git commit -m "temp: add revert-demo line"

Add-Content file.txt "`nsecond line"
git commit -am "temp: add second"

# Show log
git log --oneline -n 5

# Revert the last commit safely
git revert HEAD
# Show log - new commit created
git log --oneline -n 5

# Now demonstrate reset options (use separate branch to avoid confusion)
git checkout -b reset-demo
Add-Content file.txt "`nreset-demo"
git commit -am "reset-demo commit"

# Soft reset: keep changes staged
git reset --soft HEAD~1
git status --short
# Mixed reset: keep changes unstaged
git reset --mixed HEAD~1
git status --short
# Hard reset: destructive
git reset --hard HEAD~0  # moves back to HEAD (no-op) demonstration; to delete you can use HEAD~1 carefully
```

Presenter notes:

* Revert adds a new commit that reverses changes and is safe for published branches.
* Reset rewrites history; `--soft` preserves staged, `--mixed` unstages, `--hard` deletes working changes.
* Exercise: ask volunteers what to use to "undo but keep changes" vs "undo and remove changes".

Troubleshooting:

* If you reset hard and lose work, use `git reflog` (next demo) to recover if possible.

---

### Demo 3.2 — Log exploration power (lots of options)

```powershell
# Show compact history
git log --oneline --decorate --graph --all -n 20

# Show patch for a file across commits
git log -p -- src/lib.js -n 10

# Search commits by message
git log --oneline --grep="fix"

# Search by author (demo user)
git log --oneline --author="Demo User"

# Date range example
git log --after="2025-01-01" --before="2025-07-01" --oneline

# Compare branches (commit present in main but not in feature)
git checkout -b log-demo
git checkout main
git log log-demo..main --oneline   # commits in main not in log-demo
```

Presenter notes:

* Visual log (`--graph`) is hugely helpful in explaining merges and DAG shape.
* Exercise: ask audience to find the commit that introduced a specific line in a file using `git log -S` (see below).

Extra search:

```powershell
# find commits that added/removed a string
git log -S"hotfix" --oneline
```

---

# SECTION 2 — Advanced Git Commands (long demos)

---

## Topic: `git bisect` (full, automated + manual)

### Demo 2.1 — Manual bisect walkthrough (long)

```powershell
# Prepare a repo with reproducible "good" and "bad" commits
if (Test-Path "../bisect-work") { Remove-Item -Recurse -Force "../bisect-work" }
md ../bisect-work; Set-Location ../bisect-work
git init
"ok v1" | Out-File app.txt
git add app.txt; git commit -m "v1: ok"
"ok v2" | Out-File app.txt
git commit -am "v2: ok"
"ok v3" | Out-File app.txt
git commit -am "v3: ok"
"BUG" | Out-File app.txt
git commit -am "v4: introduces bug"
"after bug" | Out-File app.txt
git commit -am "v5: after bug"
```

Start bisect:

```powershell
git bisect start
git bisect bad HEAD
git bisect good HEAD~4
```

At each checkout, open `app.txt` (e.g., `notepad app.txt`) and decide:

```powershell
# If file shows bug:
git bisect bad

# If file is ok:
git bisect good
```

When bisect finishes, Git prints the first bad commit SHA. Finish:

```powershell
git bisect reset
```

### Demo 2.1b — Automated bisect with test script

Create a test script `test.bat` (PowerShell/Batch) that returns non-zero on bad:

```powershell
# Create test.bat
@"
@echo off
findstr /C:"BUG" app.txt >nul
if %ERRORLEVEL% EQU 0 (
  echo "bad"
  exit /b 1
) else (
  echo "good"
  exit /b 0
)
"@ | Out-File test.bat -Encoding ascii
```

Run bisect with script:

```powershell
git bisect start
git bisect bad HEAD
git bisect good HEAD~4
git bisect run .\test.bat
# Git will run the script at each step
git bisect reset
```

Presenter notes:

* Bisect reduces search from N to log2(N) tests.
* Automated `git bisect run` is powerful for testable regressions (unit/integration).
* Exercise: have audience write a failing test script and run bisect.

Troubleshooting:

* Ensure test script exits with 0 for good and non-zero for bad. Use `exit /b` in batch.

---

## Topic: Interactive Rebase & Autosquash (long)

### Demo 2.2 — Rebase interactive deep dive

```powershell
# Build a rebase demo branch
cd ../advanced-git-demo
git checkout -b rebase-demo
# Create several small commits
"line A" | Out-File rebase.txt
git add rebase.txt; git commit -m "feat: A"
"line B" | Add-Content rebase.txt
git commit -am "feat: B"
"typo fix" | Add-Content rebase.txt
git commit -am "fix: typo in B"
"cleanup" | Add-Content rebase.txt
git commit -am "chore: cleanup"
```

Now interactive rebase:

```powershell
git rebase -i HEAD~4
```

You will see a todo list in your configured editor:

```
pick <sha1> feat: A
pick <sha2> feat: B
pick <sha3> fix: typo in B
pick <sha4> chore: cleanup
```

Change to:

```
pick <sha1> feat: A
pick <sha2> feat: B
fixup <sha3> fix: typo in B
squash <sha4> chore: cleanup
```

Save & exit. Git will:

* apply fixup silently into the target commit
* ask to combine messages for squash

### Autosquash workflow:

Make explicit fixup:

```powershell
# Assume we want to fixup commit with hash HEAD~2 (example)
git commit --fixup HEAD~2
# Now run autosquash rebase for the range:
git rebase -i --autosquash HEAD~5
```

Git will auto-place fixup/squash lines under target commit so you don't manually edit ordering.

Presenter notes:

* Emphasize safety: do interactive rebase only on local branches or with team coordination.
* Show how `git log --graph --oneline --decorate` looks before and after to illustrate new SHAs.
* Exercise: have attendees create a fixup commit and run `git rebase -i --autosquash`.

Troubleshooting:

* Conflicts during rebase: resolve conflicts, `git add` files, then `git rebase --continue`. To abort: `git rebase --abort`.

---

## Topic: Hooks & Husky (practical + cross-platform)

### Demo 2.3 — Local hook example (pre-commit)

```powershell
# Show hooks directory
ls .git/hooks

# Create a pre-commit (PowerShell won't set +x; Git on Windows executes scripts when using Git Bash or when file has .cmd/.bat)
# We'll create a cross-platform Node-style hook if Node not present create a simple .cmd

# Simple Windows-compatible pre-commit (pre-commit.cmd)
@"
@echo off
REM Inspect staged content for TODO
git diff --cached --name-only | findstr /R /C:".*"
git grep -n "TODO" -- ':!node_modules' >nul 2>&1
if %ERRORLEVEL% EQU 0 (
  echo Commit blocked: TODO found
  exit /b 1
)
exit /b 0
"@ | Out-File .git/hooks/pre-commit -Encoding ascii

# Make sure it is executable if using Git Bash; on Windows, Git will run .cmd or .bat automatically
# Now test: create a TODO in a file and attempt to commit
Add-Content app.js "`n// TODO: fix this"
git add app.js
git commit -m "test: commit with TODO"  # Expect blocked by hook
```

Explain:

* On Windows, using `.cmd` or `.bat` in hooks makes it cross-compatible.
* In JS projects, prefer Husky for consistent npm-based hooks.

### Husky (Node) example

```powershell
# Only run this in a Node project; create package.json if needed
npm init -y
npm install husky --save-dev

# Initialize husky (creates .husky folder)
npx husky install
npx husky add .husky/pre-commit "npm test"
git add .husky/pre-commit
git commit -m "chore: add husky pre-commit"
```

Presenter notes:

* Hooks automate quality checks (lint/tests) before commit/push.
* Exercise: have attendees add a hook that runs `npm run lint` or a basic `dotnet test` (if dotnet present).

Troubleshooting:

* Windows CRLF issues: ensure script files use CRLF for batch or LF for bash hooks depending on environment. Use Husky to ease cross-platform issues.

---

## Topic: Dangerous Commands — safe demo & recovery

### Demo 2.4 — `git reset --hard` `git clean -df` (use disposable branch)

```powershell
# Create disposable branch
git checkout -b danger-demo

# Create a commit to "lose"
"temp content" | Out-File destroy.txt
git add destroy.txt
git commit -m "temp: will be destroyed"

# Show commit exists
git log --oneline -n 3

# Now hard reset back one commit (simulate losing last commit)
git reset --hard HEAD~1
git log --oneline -n 3   # note missing commit

# Recover with reflog (we will demonstrate reflog later)
git reflog
# Find the SHA of the "temp: will be destroyed" commit and restore
# Example (assume <sha> is the lost commit):
git checkout -b recovered <sha>
git log --oneline -n 3

# Demonstrate clean
New-Item -Path junk.txt -ItemType File
md build; New-Item -Path build\out.o -ItemType File
git clean -ndf    # dry-run (show what would be deleted)
git clean -df     # delete untracked files/directories
```

Presenter notes:

* Warn strongly before `reset --hard` and `clean -df`. Always use `-n` / dry-run first.
* Exercise: have audience run `git clean -ndf` then `git clean -df` in a disposable repo to see effect.

---

## Topic: Submodules — deep lifecycle (add, update, change, remove)

### Demo 2.5 — Add, update, commit, clone, and remove submodule (comprehensive)

> For demo speed, use a small public repo as submodule such as `https://github.com/octocat/Spoon-Knife.git` or better create a small local repo for offline demo.

**A — Add submodule**

```powershell
# In parent repo:
git submodule add https://github.com/octocat/Spoon-Knife.git extern/spoon
git commit -m "chore: add extern/spoon submodule"
git status
cat .gitmodules
```

Show `.gitmodules` content and the fact the main repo stores a *commit pointer* to the submodule (not the files).

**B — Clone parent and init submodules**

```powershell
# In a new location simulate a fresh clone
cd ..
git clone --recurse-submodules (Resolve-Path ./advanced-git-demo) advanced-git-demo-clone
cd advanced-git-demo-clone
git submodule status
# OR if clone without recurse:
git clone (Resolve-Path ./advanced-git-demo) advanced-git-demo-clone2
cd advanced-git-demo-clone2
git submodule update --init --recursive
```

Explain `git submodule status` shows commit SHA each submodule is checked out to.

**C — Update submodule to newer commit**

```powershell
# In parent repo, update submodule working directory (go into submodule)
cd extern/spoon
# fetch and checkout latest or specific branch
git fetch
git checkout main
git pull origin main
# Back to parent
cd ../../
git add extern/spoon
git commit -m "chore: update extern/spoon pointer to latest main"
```

Explain: parent repo records new submodule commit pointer — must commit that change.

**D — Submodule branch tracking / update remote**

To update submodule to latest remote by default branch automatically:

```powershell
# From parent repo
git submodule update --remote extern/spoon
# Then commit pointer update
git add extern/spoon
git commit -m "Update submodule pointer to remote HEAD"
```

**E — Removing a submodule**

```powershell
# Remove from index and tree
git submodule deinit -f -- extern/spoon   # remove from .git/config
git rm -f extern/spoon
Remove-Item -Recurse -Force .git/modules/extern/spoon   # remove metadata
git commit -m "chore: remove submodule extern/spoon"
```

**F — What happens on remote submodule changes?**

* Parent repo does NOT automatically update to latest submodule HEAD; you must:

  * go into submodule, pull/checkout new commit, then in parent `git add` that submodule and commit new pointer.
* If remote changed and another dev updates pointer and pushes parent change, pulling parent will not automatically update submodule content — run `git submodule update --init --recursive` to update working trees to recorded pointers.

Presenter notes:

* Submodules are pointers—think of them as a commit reference. Show a diagram: `Parent repo tree entry -> submodule commit SHA -> submodule repo objects`.
* Exercise: have audience add a small local submodule, update it, commit pointer change, and then in clone run `git submodule update` to sync.

Troubleshooting:

* Submodules often leave detached HEAD; after update you may want to checkout a branch inside the submodule.

---

## Topic: Advanced Git Searching (combinational queries)

### Demo 2.6 — Powerful git log filters & searching

```powershell
# Show commits matching message patterns (regex)
git log --oneline --grep="fix" -E

# By author
git log --author="Demo User" --oneline

# By date range
git log --after="2025-01-01" --before="2025-07-01" --oneline

# By file (history for a single file)
git log -p -- src/lib.js

# Searching commit content (pick/remove string)
git log -S"BUG" --oneline

# Searching commits across branches (commits in main not in feature)
git log feature/demo..main --oneline

# Combine filters: e.g., commits by author with 'fix' in message after date
git log --author="Demo" --grep="fix" --after="2024-01-01" --oneline
```

Presenter notes:

* These filters are essential for audits, code reviews, and backporting.
* Exercise: have participants find the commit that introduced a specific string via `git log -S` or `git blame`.

---

## Topic: `git cherry-pick` (long demo + conflict)

### Demo 2.7 — Cherry-pick use & conflict resolution

```powershell
# Create a branch with a single meaningful commit
git checkout -b cherry-source
"hotfix content" | Out-File hotfix.txt
git add hotfix.txt; git commit -m "fix: urgent hotfix"

# Switch to main and cherry-pick that commit
git checkout main
git cherry-pick $(git rev-parse cherry-source)

# Show history and the fact the fix appears on main
git log --graph --oneline -n 10
```

**Cherry-pick conflict demo:**

```powershell
# Create conflicting change on main to force cherry-pick conflict
git checkout main
Add-Content app.js "`n// change conflicting on main"
git commit -am "main: conflicting change"

# Try cherry-pick again (from cherry-source)
git cherry-pick $(git rev-parse cherry-source)
# Expect conflict; resolve file with notepad, then:
git add app.js
git cherry-pick --continue
```

Presenter notes:

* Cherry-pick creates a new commit on target branch with same changes but different SHA.
* Exercise: have attendees cherry-pick a commit and resolve conflict.

---

## Topic: `git reflog` (rescue & advanced recovery)

### Demo 2.8 — Reflog: find lost commits and recover

```powershell
# Create a temporary branch and delete it to create a dangling commit
git checkout -b reflog-demo
"recover this" | Out-File rescue.txt
git add rescue.txt; git commit -m "wip: rescue me"

# Delete branch (simulate mistake)
git checkout main
git branch -D reflog-demo

# Show reflog of HEAD (local history of HEAD movements)
git reflog --decorate --oneline

# Identify the commit SHA from reflog (the commit we deleted) and recreate branch
# Replace <sha> with relevant SHA from git reflog
git checkout -b recovered <sha>
git log --oneline -n 5
```

Extras:

```powershell
# Reset HEAD to earlier reflog state
git reset --hard HEAD@{3}
```

Presenter notes:

* Reflog is your "local undo buffer". Very often you can recover lost commits as long as GC hasn't pruned objects.
* Exercise: intentionally reset hard and recover via reflog.

Caveat:

* Reflog is local only — not shared between clones.

---

## Topic: `git worktree` (multiple working directories)

### Demo 2.9 — Create and use multiple worktrees

```powershell
# Make sure you have an existing branch to check out
git branch feature/x || git checkout -b feature/x

# Add a worktree pointing to a branch (creates ../worktree-feature)
git worktree add ..\worktree-feature feature/x

# List worktrees
git worktree list

# Inspect the new worktree
Set-Location ..\worktree-feature
git status
# Return to main repo
Set-Location ..\advanced-git-demo
```

Use cases to demo:

* Run long tests in one worktree while continuing development in another.
* Build or test different PRs simultaneously.

Presenter notes:

* `git worktree` uses same object database; minimal disk overhead compared to full clone.
* Exercise: have volunteers create a worktree and make separate commits in it.

---

## Topic: Gitflow vs Trunk-Based (practical comparison)

### Demo 2.10 — Simulate both strategies

**Gitflow (simplified)**

```powershell
# Create long-lived branches
git checkout -b develop
git checkout -b feature/login develop
# Merge feature into develop (non-fast-forward)
git checkout develop
git merge --no-ff feature/login -m "merge: feature/login into develop"
```

**Trunk-based (short-lived branches + frequent merges/rebase)**

```powershell
# Create short-lived branch and rebase before merging into main
git checkout -b short-fix
# do work...
git commit -am "fix: small"
# Rebase onto latest main then merge fast-forward
git checkout main
git pull origin main
git checkout short-fix
git rebase main
git checkout main
git merge --ff-only short-fix
```

Presenter notes:

* Discuss pros/cons: Gitflow suits scheduled releases & larger teams; trunk-based suits continuous delivery and smaller teams.
* Exercise: ask teams to choose a workflow for a hypothetical product and justify.

---

# FINAL — Summary, Key Takeaways, Q&A demo

### Summary commands to show entire workshop's main checks

```powershell
# Visual repo snapshot
git log --graph --oneline --decorate --all -n 30

# Show object storage & packfile details
git count-objects -v
git gc --aggressive --prune=now    # optional: repack if you want to show packfiles (be cautious)
ls .git/objects/pack
git verify-pack -v .git/objects/pack/*.idx 2>$null
```

Presenter notes:

* Reiterate internals: blobs, trees, commits, index.
* Encourage audience: try each demo in their environment and keep a disposable repo for destructive commands.

---

# Extras / Handy commands to show "power" during demos

```powershell
# Show content of a commit (diff and message)
git show <sha>

# Find commit that introduced a line (git blame + -L or pick by search)
git blame -L 1,200 -- src/lib.js
git log -S"someString" --oneline

# Compare two commits or branches with patch
git diff branchA..branchB
git range-diff main origin/main  # advanced (requires git >= 2.22)

# Show remote refs quickly
git remote show origin
```

---

# Workshop flow & timing suggestion

* Setup & Internals (15–20 min)
* Power Essentials (25–30 min)
* Advanced Commands (bisect/rebase/hooks/submodules) (40–50 min)
* Recovery, Worktrees, Workflow debate (20 min)
* Q&A + Live recovery challenge (15 min)

You can shrink or extend each demo depending on time.

---

# Final presenter tips

* Always run destructive demos on disposable branches or separate working copies.
* Keep `git status` and `git log --graph --oneline --decorate --all` visible frequently — they are the “state map”.
* When showing internal object commands (`git cat-file`), explain the mapping between human files and stored objects.
* Use the `--no-ff` merge to show merge commits when illustrating Gitflow.
* Encourage audience to experiment: provide a GitHub/GitLab sample repo or host the demo repo for them to clone.

---

If you want, I’ll next:

1. Produce **one combined PowerShell script** that runs through all non-destructive demos and prints explanations (you can run it in front of the audience).
2. Produce **slide-ready snippet blocks** (one slide per demo) including an ASCII diagram and presenter notes.
3. Create a **handout PDF** summarizing commands + exercises for attendees.

Which of 1/2/3 do you want me to generate right now?
