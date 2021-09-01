---
title: 'Git 101'
tags: GIT
---

Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.<!--more--> Git gots lot of available advanced commands. This being said, this blog is a curated list of the command you'd need to survive in the harsh world of computer sciences.

## Create branch
```bash
git branch <branch_name>
```

## Rename branch
```bash
git branch -m <new_branch_name> # rename current branch to new_branch_name
git branch -m <old_branch_name> <new_branch_name> # rename old_branch_name branch to new_branch_name
```

## Checkout to a branch
```bash
git checkout <branch_name> # change current followed branch
git checkout -b <branch_name> # checkout to a new branch and create it if it doesn't exists 
```

## Pull code
```bash
git pull # pull from current branch
git pull origin <branchname> # pull from a remote branch
```

## Add code
```bash
git add <filename> # Add a specific filename 
git add -A # Add all modified code
```

## Commit modifications
```bash
git commit -m 'Adding this file' #  Add a commit
```

## Push code
```bash
git push # push all local commits to remote branch
git push origin <branchname> # push all local commits to an alternative remote branch
```

## History log
```bash
git log # retrieve local commit history
```

## Reset to older state
```bash
git reset --soft <commit_hash> # restore state to commit and keep in between modified files
git reset --hard <commit_hash> # restore state to commit and delete in between modified files 

git reset --hard origin/<branchname> # reset to a remote branch state
```

## Stash
```bash
git stash #  record the current state of the working directory and the index
git stash apply # apply stored stashed state onto current code
```

## Cherry-pick
```bash
git cherry-pick <commit_hash>
```

## Squash commit
```bash
git log # found the commit hash before your 1st modification
git reset --soft <commit_hash> # reset to the state before you added your modifications
git stash
git pull
git stash apply
git add -A
git commit -m "squahed commits into one commit"
git push
```

Git tutorial available [here](https://www.atlassian.com/fr/git/tutorials/using-branches/git-checkout)