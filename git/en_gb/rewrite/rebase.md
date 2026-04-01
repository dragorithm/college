# Rebase

[index](./index.md)

A simple `git commit --amend` can only modify the most recent commit.
    If you want to change other commits, `git rebase` is the recommended approach.

## Rebase HEAD

### Basic Syntax

```bash
git rebase -i HEAD~N
```

`N`: the number of commits to go back.

### Usage

For examples:

```bash
git log --oneline -6
cb26643 (HEAD -> master) [feat.add](notes): add new note on Git editor
bb1205c [feat.del](notes): remove duplicate notes
c066c0e [feat.up](notes): make user configuration more comprehensive and remove redundant steps
3dcbfba [refactor](notes): refactor notes by consolidating safe.directory under config
2af8d64 [feat.add](notes): add new note about git config
05f6deb [feat.add](college): add new Linux practical task on disk management
```

- To change `cb26643`: `git rebase -i HEAD~1`
- To change `bb1205c`: `git rebase -i HEAD~2`
- To change `c066c0e`: `git rebase -i HEAD~3`
- To change `3dcbfba`: `git rebase -i HEAD~4`
- ...

*Note*: To change the first commit(`cb26643`), you don't need rebase; simply use [amend].

### Example

To change `bb1205c`

```bash
git rebase -i HEAD~2
```

The editor will open with:

```txt
pick bb1205c # [feat.del](notes): remove duplicate notes
pick cb26643 # [feat.add](notes): add new note on Git editor

# Rebase c066c0e..cb26643 onto c066c0e (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
#                       updated at the end of the rebase
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```

Ignore the lines starting with `#`.
    Focus on the `pick` lines, and change `bb1205c` from `pick` to `edit`:

```txt
edit bb1205c # [feat.del](notes): remove duplicate notes
pick cb26643 # [feat.add](notes): add new note on Git editor
```

Then amend the commit(see [amend]):

```bash
git commit --amend
```

After editing, the commit remains at `HEAD`.
    Use the following command to continue:

```bash
git rebase --continue
```

## Rebase by Commit ID

### Basic Syntax

```bash
git rebase -i id^
```

### Usage

Examples:

```bash
git log --oneline -6
cb26643 (HEAD -> master) [feat.add](notes): add new note on Git editor
bb1205c [feat.del](notes): remove duplicate notes
c066c0e [feat.up](notes): make user configuration more comprehensive and remove redundant steps
3dcbfba [refactor](notes): refactor notes by consolidating safe.directory under config
2af8d64 [feat.add](notes): add new note about git config
05f6deb [feat.add](college): add new Linux practical task on disk management
```

- To change `cb26643`: `git rebase -i cb26643^`
- To change `bb1205c`: `git rebase -i bb1205c^`
- To change `c066c0e`: `git rebase -i c066c0e^`
- To change `3dcbfba`: `git rebase -i 3dcbfba^`
- ...

*Note*: To change the first commit, simply use [amend].
    On Windows CMD, `^` is interpreted, so you need to escape it: `git rebase -i cb26643^^`

This differ slightly from `rebase HEAD`, but the other steps are the same.
    The main advantage: **no need to count commits manually**.

## Edit Todo

If you make a mistake in the rebase todo list, you can edit it again:

```bash
git rebase --edit-todo
```

Example: you accidentally wrote `editor` instead of `edit`.

## Reword

Instead of changing `pick` to `edit`, you can change it to `reword`.
    After saving and closing, Git will automatically reopen the editor for you to edit the commit message.
    Once done, the process end - no need for `amend` or `continue`.
    This method is best suited for purely changing the **commit message.**

[amend]:./amend.md
