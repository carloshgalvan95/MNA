# Git Cheat Sheet


## SETUP AND CONFIG

### Create a new repository
    git init

### Clone a repository (with submodules)
    git clone --recurse-submodules <url>

### Clone a repository (without submodules, add them later)
    git clone <url>
    git submodule update --init --recursive

### Add a remote
    git remote add origin <url>

### View remotes
    git remote -v

### Set user name and email (per repo)
    git config user.name "Your Name"
    git config user.email "your@email.com"

### Set user name and email (global)
    git config --global user.name "Your Name"
    git config --global user.email "your@email.com"

### View all config
    git config --list


---


## BASIC WORKFLOW

### Check repo status
    git status

### Check short status
    git status -s

### Stage a file
    git add <file>

### Stage all changes
    git add .

### Stage parts of a file interactively
    git add -p <file>

### Unstage a file (keep changes)
    git restore --staged <file>

### Commit staged changes
    git commit -m "message"

### Commit with multi-line message
    git commit -m "title" -m "body details"

### View unstaged changes
    git diff

### View staged changes
    git diff --cached

### View commit history (compact)
    git log --oneline

### View commit history (detailed)
    git log

### View history with graph
    git log --oneline --graph --all

### View changes in a specific commit
    git show <commit-hash>

### Scenario: I modified files but want to discard all local changes
    git restore .

### Scenario: I want to discard changes to a specific file
    git restore <file>

### Scenario: I accidentally staged a file I didn't mean to
    git restore --staged <file>
    # The file stays modified in your working directory, just removed from staging


---


## BRANCHING AND MERGING

### List all branches
    git branch -a

### Create a new branch
    git branch <branch-name>

### Switch to a branch
    git switch <branch-name>
    # or: git checkout <branch-name>

### Create and switch to a new branch
    git switch -c <branch-name>
    # or: git checkout -b <branch-name>

### Merge a branch into current branch
    git merge <branch-name>

### Delete a branch (already merged)
    git branch -d <branch-name>

### Delete a branch (force, even if not merged)
    git branch -D <branch-name>

### Rename current branch
    git branch -m <new-name>

### Rebase current branch onto another
    git rebase <branch-name>

### Abort a merge in progress
    git merge --abort

### Abort a rebase in progress
    git rebase --abort

### Scenario: I committed to the wrong branch
    # Option A: Move the last commit to the correct branch
    git log --oneline -1                  # note the commit hash
    git reset --soft HEAD~1               # undo commit, keep changes staged
    git stash                             # stash the changes
    git switch <correct-branch>           # switch to the right branch
    git stash pop                         # apply the changes
    git add .
    git commit -m "message"

    # Option B: Cherry-pick (if you already switched away)
    git switch <correct-branch>
    git cherry-pick <commit-hash>         # apply the commit here
    git switch <wrong-branch>
    git reset --hard HEAD~1               # remove it from wrong branch

### Scenario: I need to resolve a merge conflict
    git merge <branch>                    # triggers conflict
    # 1. Open conflicted files -- look for <<<<<<< / ======= / >>>>>>>
    # 2. Edit files to keep the correct code, remove conflict markers
    # 3. Stage resolved files:
    git add <resolved-file>
    # 4. Complete the merge:
    git commit                            # git will auto-generate merge message

### Scenario: I want to see what changed between two branches
    git diff <branch-a>..<branch-b>
    git log <branch-a>..<branch-b> --oneline


---


## REMOTE OPERATIONS

### Fetch latest from remote (without merging)
    git fetch origin

### Pull latest (fetch + merge)
    git pull origin <branch>

### Pull with rebase instead of merge
    git pull --rebase origin <branch>

### Push to remote
    git push origin <branch>

### Push and set upstream tracking
    git push -u origin <branch>

### Delete a remote branch
    git push origin --delete <branch-name>

### View tracking branches
    git branch -vv

### Scenario: Push was rejected because remote has new commits
    git pull origin main                  # pull and merge remote changes
    git push origin main                  # now push succeeds

### Scenario: I want to see what the remote has without changing anything
    git fetch origin
    git log HEAD..origin/main --oneline   # see incoming commits

### Scenario: I want to overwrite my local branch with remote (DESTRUCTIVE)
    git fetch origin
    git reset --hard origin/main          # WARNING: discards all local changes


---


## SUBMODULES

### Add a submodule
    git submodule add <url> <path>

### Initialize submodules (after cloning)
    git submodule update --init --recursive

### View submodule status
    git submodule status

### Pull latest for main repo AND all submodules
    git pull --recurse-submodules

### Make recursive submodule pull the default
    git config --global submodule.recurse true

### Update all submodules to their tracked commits
    git submodule update --recursive

### Scenario: A submodule has new commits on its remote and I want to update it
    cd <submodule-dir>                    # enter the submodule
    git pull origin main                  # pull latest in submodule
    cd ..                                 # go back to main repo root
    git add <submodule-dir>               # stage the updated pointer
    git commit -m "Update submodule"      # commit the new reference
    git push                              # push main repo

### Scenario: Someone else updated the submodule reference and I need to sync
    git pull --recurse-submodules         # pulls main repo + updates submodules
    # or:
    git pull
    git submodule update --init --recursive

### Scenario: Submodule is in "detached HEAD" state
    cd <submodule-dir>
    git checkout main                     # switch to the branch you want
    git pull                              # pull latest
    cd ..

### Scenario: I cloned a repo but submodules are empty directories
    git submodule update --init --recursive

### Scenario: I want to see what commit each submodule points to
    git submodule status


---


## UNDOING THINGS

### Undo last commit, keep changes staged
    git reset --soft HEAD~1

### Undo last commit, keep changes unstaged
    git reset HEAD~1
    # or: git reset --mixed HEAD~1

### Undo last commit, discard changes entirely (DESTRUCTIVE)
    git reset --hard HEAD~1

### Amend the last commit message (not yet pushed)
    git commit --amend -m "new message"

### Amend the last commit to include new staged files (not yet pushed)
    git add <forgotten-file>
    git commit --amend --no-edit

### Revert a commit (creates a new "undo" commit, safe for shared branches)
    git revert <commit-hash>

### Scenario: I want to undo my last commit but keep all my changes
    git reset --soft HEAD~1
    # Changes are back in staging area, ready to re-commit differently

### Scenario: I pushed a bad commit and need to undo it safely
    git revert <commit-hash>              # creates a new commit that undoes it
    git push                              # safe for shared branches

### Scenario: I want to completely discard the last 3 commits (DESTRUCTIVE)
    git reset --hard HEAD~3
    # WARNING: This permanently deletes those commits and their changes

### Scenario: I amended a commit but I already pushed it
    # You need a force push (coordinate with team first!)
    git push --force-with-lease           # safer than --force

### Scenario: I want to undo a file to how it was in a specific commit
    git restore --source <commit-hash> -- <file>


---


## STASHING

### Stash current changes
    git stash

### Stash with a descriptive message
    git stash push -m "description of changes"

### List all stashes
    git stash list

### Apply most recent stash (keep it in stash list)
    git stash apply

### Apply most recent stash and remove it from stash list
    git stash pop

### Apply a specific stash
    git stash apply stash@{2}

### Drop a specific stash
    git stash drop stash@{0}

### Clear all stashes (DESTRUCTIVE)
    git stash clear

### Stash including untracked files
    git stash -u

### Scenario: I need to switch branches but have uncommitted changes
    git stash                             # save changes
    git switch <other-branch>             # do your work there
    # ... work on other branch ...
    git switch <original-branch>          # come back
    git stash pop                         # restore your changes

### Scenario: I stashed something a while ago and forgot what it was
    git stash list                        # see all stashes with messages
    git stash show stash@{0}              # see file-level summary
    git stash show -p stash@{0}           # see full diff of changes


---


## ADVANCED

### Cherry-pick a commit from another branch
    git cherry-pick <commit-hash>

### Cherry-pick without committing (stage only)
    git cherry-pick --no-commit <commit-hash>

### Create a tag
    git tag v1.0.0

### Create an annotated tag
    git tag -a v1.0.0 -m "Release version 1.0.0"

### Push tags to remote
    git push origin --tags

### List tags
    git tag -l

### Delete a tag locally
    git tag -d v1.0.0

### Delete a tag on remote
    git push origin --delete v1.0.0

### Remove untracked files (dry run first)
    git clean -n                          # preview what will be deleted
    git clean -fd                         # actually delete untracked files and dirs

### Blame -- see who changed each line
    git blame <file>

### Search commit messages
    git log --grep="search term"

### Search code changes
    git log -S "function_name"            # find commits that added/removed this string

### View a file from another branch without switching
    git show <branch>:<file-path>

### Scenario: I need one specific commit from another branch
    git log <other-branch> --oneline      # find the commit hash
    git cherry-pick <commit-hash>         # apply it to current branch

### Scenario: I want to clean up my commit history before pushing
    git rebase -i HEAD~3                  # interactive rebase on last 3 commits
    # In the editor: pick, squash, reword, drop commits as needed
    # Save and close to apply

### Scenario: I want to find which commit introduced a bug
    git bisect start
    git bisect bad                        # current commit has the bug
    git bisect good <known-good-commit>   # this commit was fine
    # Git will checkout commits for you to test
    # For each, run your test and then:
    git bisect good                       # or: git bisect bad
    # When done:
    git bisect reset

### Scenario: I want to temporarily go back to an old commit to test something
    git checkout <commit-hash>            # detached HEAD state
    # ... test things ...
    git switch main                       # go back to your branch


---


## QUICK REFERENCE -- COMMON WORKFLOWS

### Daily workflow
    git pull --recurse-submodules         # start by syncing
    # ... make changes ...
    git add .
    git commit -m "description"
    git push

### Feature branch workflow
    git switch -c feature/my-feature      # create feature branch
    # ... make changes, commit as you go ...
    git push -u origin feature/my-feature # push feature branch
    # Create pull request on GitHub
    # After merge, clean up:
    git switch main
    git pull
    git branch -d feature/my-feature

### Updating submodules after pull
    git pull --recurse-submodules
    # or if you forgot --recurse-submodules:
    git submodule update --init --recursive
