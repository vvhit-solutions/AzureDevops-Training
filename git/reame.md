# Git Commands Cheat Sheet

## Table of Contents
1. [Basic Git Commands](#basic-git-commands)
2. [Intermediate Git Commands](#intermediate-git-commands)
3. [Advanced Git Commands](#advanced-git-commands)
4. [Expert Git Commands](#expert-git-commands)

---

## Basic Git Commands
These are essential commands for beginners to get started with Git.

### 1. Git Configuration
- **`git config --global user.name "Your Name"`**  
  *Set up your Git username.*
- **`git config --global user.email "your-email@example.com"`**  
  *Set up your Git email.*
- **`git config --global core.editor "vim"`**  
  *Set the default editor for Git.*
- **`git config --list`**  
  *View the current configuration.*
- **`git config --global alias.co checkout`**  
  *Set an alias for a Git command.*

### 2. Initializing and Cloning Repositories
- **`git init`**  
  *Initialize a new Git repository.*
- **`git clone <repository-url>`**  
  *Clone an existing repository.*
- **`git clone --depth=1 <repository-url>`**  
  *Clone a repository with a shallow history.*

### 3. Staging and Committing
- **`git status`**  
  *Check the status of your repository.*
- **`git add <file>`**  
  *Add a file to the staging area.*
- **`git add .`**  
  *Add all files to the staging area.*
- **`git commit -m "commit message"`**  
  *Commit staged changes with a message.*
- **`git commit --amend -m "new message"`**  
  *Modify the last commit message.*

### 4. Branching
- **`git branch`**  
  *List all branches.*
- **`git branch <branch-name>`**  
  *Create a new branch.*
- **`git checkout <branch-name>`**  
  *Switch to another branch.*
- **`git checkout -b <branch-name>`**  
  *Create and switch to a new branch.*
- **`git branch -d <branch-name>`**  
  *Delete a branch.*
- **`git branch -m <old-name> <new-name>`**  
  *Rename a branch.*

### 5. Pushing and Pulling
- **`git push origin <branch-name>`**  
  *Push changes to a remote repository.*
- **`git pull origin <branch-name>`**  
  *Pull updates from a remote repository.*
- **`git fetch`**  
  *Fetch changes from a remote repository.*
- **`git push --force`**  
  *Force push changes (use with caution).* 

---

## Intermediate Git Commands
These commands help manage repositories efficiently.

### 1. Viewing History
- **`git log`**  
  *View commit history.*
- **`git log --oneline --graph --all`**  
  *View history in a compact format with branches.*
- **`git diff`**  
  *Show differences between files.*
- **`git blame <file>`**  
  *Show who changed each line of a file.*
- **`git shortlog -sn`**  
  *Show commit statistics by author.*

### 2. Merging and Rebasing
- **`git merge <branch-name>`**  
  *Merge another branch into the current branch.*
- **`git rebase <branch-name>`**  
  *Reapply commits on top of another base commit.*
- **`git merge --squash <branch-name>`**  
  *Merge changes without creating a merge commit.*

### 3. Undoing Changes
- **`git reset --soft HEAD~1`**  
  *Undo the last commit but keep changes staged.*
- **`git reset --hard HEAD~1`**  
  *Undo the last commit and remove changes.*
- **`git revert <commit-hash>`**  
  *Create a new commit that undoes a specific commit.*
- **`git checkout -- <file>`**  
  *Discard changes in a file.*

### 4. Stashing Changes
- **`git stash`**  
  *Temporarily save uncommitted changes.*
- **`git stash apply`**  
  *Apply the most recent stash.*
- **`git stash list`**  
  *View all stashes.*
- **`git stash drop`**  
  *Delete the latest stash.*

---

## Advanced Git Commands
These commands are useful for resolving conflicts and optimizing workflows.

### 1. Working with Tags
- **`git tag <tag-name>`**  
  *Create a new tag.*
- **`git tag -a <tag-name> -m "message"`**  
  *Create an annotated tag.*
- **`git push origin --tags`**  
  *Push all tags to a remote repository.*
- **`git tag -d <tag-name>`**  
  *Delete a tag locally.*

### 2. Handling Conflicts
- **`git merge --abort`**  
  *Abort a conflicted merge.*
- **`git cherry-pick <commit-hash>`**  
  *Apply a specific commit from another branch.*
- **`git bisect start`**  
  *Find the commit that introduced a bug.*
- **`git reset --merge`**  
  *Reset merge conflicts.*

---

## Expert Git Commands
These commands offer more control over Git repositories.

### 1. Rewriting History
- **`git rebase -i HEAD~n`**  
  *Interactively rebase the last `n` commits.*
- **`git commit --amend`**  
  *Modify the last commit message.*
- **`git filter-branch --tree-filter '<command>' HEAD`**  
  *Rewrite history by applying a command to all commits.*
- **`git reflog`**  
  *Show a log of all operations affecting HEAD.*

### 2. Working with Submodules
- **`git submodule add <repository-url>`**  
  *Add a submodule.*
- **`git submodule update --init --recursive`**  
  *Initialize and update submodules.*
- **`git submodule foreach git pull origin main`**  
  *Update all submodules.*

### 3. Performance Optimization
- **`git gc --prune=now`**  
  *Clean up unnecessary files.*
- **`git fsck`**  
  *Verify integrity of the repository.*
- **`git repack -a -d`**  
  *Repack repository to optimize space.*
- **`git prune`**  
  *Remove unreachable objects from the database.*

---

## Conclusion
This guide provides a structured overview of essential, advanced, and expert-level Git commands. Mastering these commands will help improve your workflow and repository management skills!

