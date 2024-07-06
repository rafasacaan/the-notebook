# git-notes

## handy commands to explore commits

Check *unstaged* file with the last commit 
> git diff my_file.py

Check the *staged* difference w/respect (-r) t the last commit
> git diff -r HEAD my_file.py

Compare all staged files with the last comiit
> git diff -r HEAD

Or to the previous to the last commit
> git diff -r HEAD~1

We can also see the file version for a given commit (use first 6-8 characters of the hash)
> git show 36b761

or w/respect to the before last commit
> git show HEAD~1

or between commits
> git show HEAD~1 HEAD~2

## unstage files - reset
unstage a staged file 
> git reset HEAD my_file.py

remove all files from staging area
> git reset HEAD

reverse an unstaged file to the version of the last commit
> git checkout -- my_file.py

to reverse all unstaged files to last commit
> git checkout .

## to revert to an old version of a file
> git checkout dc9d84 file.txt

## to revert to all files to an old version
> git checkout dc9d84

## list untracked files
> git clean -n

and delete them with
> git clean -f

## remotes and fetch + merge / pull
check all remotes
> git remote -v

git fetch only brings remote repo content to local, then, we need to synchronize both contents
> git fetch origin main
> git merge origin main

Here, we can also make:
> git diff origin main
to checkout the differences between the remote and local repos.

A fast-forward type of merge indicates that the local repo was behind the remote, and the merge caught up to the remote version.
Its often the case that the remote is *ahead* of the local repo. To simplify, we can do both operations in one with:
> git pull origin main
In that way we pulled from origin to local main branch.

## pushing a remote
the opposite of pull.
> git push remote local_main

## push/pull workflow
- pull from repo
- work locally
- commit changes multiple times
- push updated repo

