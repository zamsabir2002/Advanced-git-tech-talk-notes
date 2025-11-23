- `git stash` (not really advanced)

- `git checkout -` to go back to the previous branch

- `git commit --ammend` for just modifying the latest commit any further than that then we choose interactive rebase

- `git config --global alias.ac "commit -am"`
    - makes an alias "ac" that runs the "commit -am" command which is short for add and commit at the same time (e.g. use `git ac "My commit"`)

- `git comment --amend -m "ammended commit"` (Only for when code not pushed remote because for that we need `push force` which can revert all other commits not made by you)
    - TO add to the latest commit instead of creating new one
    - For same message add `git comment --amend --no-edit`

- `git revert the-commit`
    - To revert back to a stable commit

- `git log` but very hard to read
    - `git log --oneline` shows only one liners for each commit (the hash the commit message)
    - also `git log -p './filename_with_relative_path.any` to view both the commits and the changes made on the file
    - `git log --graph --oneline --decorate` beautiful layout

- `git bisect` (To go step by step to a stable commit)
    - `git bisect start` then either `git besect good` to move on to the next commit or `git bisect bad`

- `git rebase branch --interactive`
    - pulls up a file to ask what to do with commits
    - `pick`to use that commit
    - `squash` to combine or squash everythin into the original commit 
    - then another file to add the new message for the squashed commit
    - `fixup`
    - `git commit --fixup hash` or `git commit --squash hash` to tell git in advanced what to do on rebase with `git rebase -i --autosquash` 
    - `git rebase -i HEAD~3` (Meaning you want to interact with the 3 latest commits) (or add the hash of the commit till where you want to go)

- `git hooks` ( run code before or after events )
    - inside the `.git` folder look for `hooks` folder
    - `husky` for JS and e.g. create a script to validate and/or lint with every commit

- `Destruction`
    - `git reset --hard origin/master` (would just wipe everything on local or current branch and replace with said branch)
    - `git clean -df` (cleans the build artifacts and all untracked files)

- `git submodule add [link-to-remote-repo]`
    - For adding sub modules to your system so that they are managed externally from the repo instead of making your repo bulky.
    - After running add you need to commit the code.
    - `git clone` the submodules would be empty. So after that we run `git submodule update --init --recursive` also we can add option with git clone by doing `git clone --recurse-submodules [repo-to-copy]`

- git searches (by date, message, author, file, branch) (can also be combined)
    - Date: `git log --after="2025-1-1"` || `git log --after="2025-1-1" --before="2025-7-1"` (so all between january and july)
    - message: `git log --grep="keyword"` (can also accept regex)
    - author: `git log --author="Areez"`
    - file: `git log -- README.md` (all commits related to this) (-- to not confuse branch with file)
    - branch (very handy to check other branches while in your branch): `git log feature/login..main` (all commits that are in main but not in your feature)