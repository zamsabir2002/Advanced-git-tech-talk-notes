---
marp: true
theme: default
class: lead
paginate: true
backgroundColor: #0d1117
color: #ffffff
---

# **Advanced Git**  
### *Mastering Git Beyond Basics*  
#### Presented by Zameet Sabir

---

# **Agenda**

- What makes Git "advanced"
- Stashing & cleanup tools  
- Rewriting history (amend, rebase, fixups, autosquash)  
- Powerful logging & searching  
- Working with submodules  
- Git hooks  
- Destructive commands (reset, clean)  
- Recovery tools  
- Real-world workflows  
- Q/A

---

# **Understanding “Advanced Git”**

Advanced Git =  
Working with Git intentionally, not accidentally.

You’ll learn how to:
- Clean history  
- Undo anything  
- Debug commits  
- Search professionally  
- Automate actions  
- Manage complex repos  

---

# **`git stash`**  
### *Temporarily park your work*

Use when:
- You are mid-feature  
- Need to switch branches quickly  
- Cannot commit messy/unbuilt code

```

git stash
git stash pop

```

Not truly “advanced”, but very useful.

---

# **`git checkout -`**  
### Switch to the previous branch

Like Alt+Tab for Git branches.

```

git checkout -

```

---

# **Amending Commits**

# `git commit --amend`

Use when:
- You want to modify the latest commit  
- Fix message or include missing files  
- ONLY SAFE **before pushing to remote**

```

git commit --amend -m "Updated commit"

```
---

Keep same message:

```

git commit --amend --no-edit

```

---

# ⚠️ **Why amend only before pushing?**

Because amending rewrites commit history → new SHA.

If pushed → you’ll need:

```

git push --force

```

And that risks deleting teammates' commits.

---

# **Git Aliases**

Create shortcuts:

```

git config --global alias.ac "commit -am"

```

Usage:

```

git ac "My commit"

```

This will:
- Add all tracked files  
- Commit in one step  

---

# **Reverting Commits**

# `git revert <hash>`

Creates a new commit that undoes the changes of the specified commit.

Safe because:
- Does NOT rewrite history  
- Good for shared branches  

---

# **Viewing Commit History**

## `git log` (default: too verbose)

Better ways:

```

git log --oneline

```
```

git log -p ./file.txt

```
---
```

git log --graph --oneline --decorate

```

Produces a beautiful graph:

```

* f291 Fix crash
* 81bf Add validation
  | * 12ab Feature branch work
  |/
* 008c Initial commit

```

---

# **Git Bisect**  
### *Find the commit that introduced a bug*

Binary search across history.

```

git bisect start
git bisect bad         # current commit is bad
git bisect good <hash> # known good point

```
---
Then mark each step:

```

git bisect good
git bisect bad

```

Git finds the exact breaking commit.

---

# **Interactive Rebase**

## `git rebase -i <branch>`
## `git rebase -i HEAD~3`

Allows you to:
- Reorder commits  
- Squash commits  
- Rewrite messages  
- Remove commits  
- Clean up your history  
- Prepare PR-ready code  

---

# **Rebase Keywords**

- `pick` — keep the commit  
- `squash` — merge commit into previous + edit message  
- `fixup` — merge into previous without keeping message  
- `drop` — delete commit  
- `reword` — edit commit message  

---

# **Autosquash + Fixup**

Tell Git in advance what to do later:

```

git commit --fixup <hash>
git commit --squash <hash>

```

Then:

```

git rebase -i --autosquash HEAD~5

```

Git will automatically reorder & group commits for squashing.

---

# **Git Hooks**  
### Automating Git Actions

Located at:

```

.git/hooks/

```

Examples:
- `pre-commit` → lint code  
- `pre-push` → prevent pushing to main  
- `commit-msg` → enforce message style  

JS devs use **Husky** for easy hooks.

---

# **Destruction Commands**

## ⚠️ Be careful with these

### Reset local branch to remote:

```

git reset --hard origin/main

```

Wipes ALL local changes.

---

# **Delete untracked files**

Including build folders:

```

git clean -df

```

Very powerful. Very destructive.

---

# **Git Submodules**

Allow embedding another Git repo inside yours.

Add:

```

git submodule add <repo-url>

```

After cloning a repo with submodules:

```

git submodule update --init --recursive

```

---

Clone with submodules included:

```

git clone --recurse-submodules <url>

```

---

# **Advanced Git Searches**

### By date:

```

git log --after="2025-1-1"
git log --after="2025-1-1" --before="2025-7-1"

```

### By message (supports regex):

```

git log --grep="fix"

```

---

### By author:

```

git log --author="Zameet"

```

### All commits that changed a file:

```

git log -- README.md

```

### Branch comparison:

```

git log feature/login..main

```

---

# **Bonus Tool: `git reflog`**  
### *The REAL safety net*

Shows all moves of HEAD:

```

git reflog

```

Recover ANYTHING:

```

git checkout <hash>

```

Git rarely loses data — reflog saves lives.

---

# **Bonus: Git Worktrees**

Multiple working directories on the same repo.

```

git worktree add ../temp feature/login

```

Work on two branches simultaneously.

---

# **Final Thoughts**

Advanced Git is about:
- Clean commits  
- Safe collaboration  
- Undoing mistakes  
- Debugging efficiently  
- Understanding history  
- Automating workflows  

Practice these and Git becomes predictable, not scary.

---

# **Q / A**  
### Thank you!
