# Rewrite the Most Recent Commit

[index](./index.md)

```bash
git commit --amend
```

This command is commonly used to:

- Modify the commit message
- Include files that were missed during `git add`

*Note*: Before running this command, you must confirm the contents of the **staging area** to avoid accidentally committing files that should not have been added.

## Example

Edit the commit:

```bash
git commit --amend
```

The editor will open a similar interface:

```txt
[feat.del](notes): remove duplicate notes

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Apr 1 11:36:38 2026 +0800
#
# interactive rebase in progress; onto c066c0e
# Last command done (1 command done):
#    edit bb1205c # [feat.del](notes): remove duplicate notes
# Next command to do (1 remaining command):
#    pick cb26643 # [feat.add](notes): add new note on Git editor
# You are currently editing a commit while rebasing branch 'master' on 'c066c0e'.
#
# Changes to be committed:
#   deleted:    git/en_gb/config.md
#   deleted:    git/zh_cn/config.md
#
```

Most of the lines starting with `#` can be ignored.
    Focus on the lines without `#`, as there are the **commit message**.
    For example, you can change it to:

```txt
[feat.del](notes): remove duplicate notes with vscode
```

Then save and close the editor

As you can see, the **commit message** has been updated, and the **hash value** has also changed.

```bash
commit 18a2c9e9510a2872dbfa5ef3fec51695a4d905f3
Author: gzs <dragon@dragorithm.top>
Date:   Wed Apr 1 11:36:38 2026 +0800

    [feat.del](notes): remove duplicate notes with vscode
```
