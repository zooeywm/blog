+++
title = 'Git Tips'
date = 2023-12-15T23:55:32+08:00
summary = 'Git Tips'
tags = ["Git"]
+++

# Git Rebase

I used to use a casual message when I commit my code, but I found it is not a good habit. So I want to delete the commit. I found the following command to do it.

First, use git log to find the commit hash of the commit you want to change, and run the following command:

```
git rebase -i <casual-message-commit-hash>~2
```

Then, git will open your todo editor, and you can change each commit's action. There we just need to find the hash of the commit which we want to delete, and change the action in front of it from pick to squash.

Then save and exit.
This action will make it squash with it's parent and you will edit the new squashed commit message. Finally, use git push -f to push your code to remote.

Or you can simply use fixup or reword instead of squash

Here is all commands in `rebase`:

```
Commands:
p, pick <commit> = use commit
r, reword <commit> = use commit, but edit the commit message
e, edit <commit> = use commit, but stop for amending
s, squash <commit> = use commit, but meld into previous commit
f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
                   commit's log message, unless -C is used, in which case
                   keep only this commit's message; -c is same as -C but
                   opens the editor
x, exec <command> = run command (the rest of the line) using shell
b, break = stop here (continue rebase later with 'git rebase --continue')
d, drop <commit> = remove commit
l, label <label> = label current HEAD with a name
t, reset <label> = reset HEAD to a label
m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
        create a merge commit using the original merge commit's
        message (or the oneline, if no original merge commit was
        specified); use -c <commit> to reword the commit message
u, update-ref <ref> = track a placeholder for the <ref> to be updated
                      to this position in the new commits. The <ref> is
                      updated at the end of the rebase

These lines can be re-ordered; they are executed from top to bottom.

If you remove a line here THAT COMMIT WILL BE LOST.

However, if you remove everything, the rebase will be aborted.

```
