# Rebase

[index](./index.md)

简单的 `git commit --amend` 只能修改最近的提交。
    如果你想修改其他提交，建议使用 `git rebase`。

## rebase HEAD

### 基本语法

```bash
git rebase -i HEAD~N
```

N：回溯至第`N`条

### 使用方法

例如：

```bash
git log --oneline -6
cb26643 (HEAD -> master) [feat.add](notes): add new note on Git editor
bb1205c [feat.del](notes): remove duplicate notes
c066c0e [feat.up](notes): make user configuration more comprehensive and remove redundant steps
3dcbfba [refactor](notes): refactor notes by consolidating safe.directory under config
2af8d64 [feat.add](notes): add new note about git config
05f6deb [feat.add](college): add new Linux practical task on disk management
```

- 要更改`cb26643`: `git rebase -i HEAD~1`
- 要更改`bb1205c`: `git rebase -i HEAD~2`
- 要更改`c066c0e`: `git rebase -i HEAD~3`
- 要更改`3dcbfba`: `git rebase -i HEAD~4`
- ...

*注意*：要修改第一个提交（`cb26643`），无需使用 rebase；只需使用 [amend]。

### 示例

要修改 `bb1205c`

```bash
git rebase -i HEAD~2
```

编辑器将打开并显示：

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

忽略以 `#` 开头的行。
    关注 `pick` 行，并将 `bb1205c` 中的 `pick` 改为 `edit`：

```txt
edit bb1205c # [feat.del](notes): remove duplicate notes
pick cb26643 # [feat.add](notes): add new note on Git editor
```

然后修改提交（参见 [amend]）：

```bash
git commit --amend
```

编辑完成后，提交仍保留在 `HEAD`。
    使用以下命令继续：

```bash
git rebase --continue
```

## 按提交 ID 进行 Rebase

### 基本语法

```bash
git rebase -i id^
```

### 使用方法

示例：

```bash
git log --oneline -6
cb26643 (HEAD -> master) [feat.add](notes): add new note on Git editor
bb1205c [feat.del](notes): remove duplicate notes
c066c0e [feat.up](notes): make user configuration more comprehensive and remove redundant steps
3dcbfba [refactor](notes): refactor notes by consolidating safe.directory under config
2af8d64 [feat.add](notes): add new note about git config
05f6deb [feat.add](college): add new Linux practical task on disk management
```

- 要更改`cb26643`: `git rebase -i cb26643^`
- 要更改`bb1205c`: `git rebase -i bb1205c^`
- 要更改`c066c0e`: `git rebase -i c066c0e^`
- 要更改`3dcbfba`: `git rebase -i 3dcbfba^`
- ...

*注意*：若要修改第一个提交，只需使用 [amend]。
    在 Windows 命令提示符中，`^` 会被解释，因此需要进行转义：`git rebase -i cb26643^^`

这与 `rebase HEAD` 略有不同，但其余步骤相同。
    主要优势：**无需手动计数提交**。

## 编辑 Todo

如果你在 rebase todo 列表中写错了，可以再次编辑它：

```bash
git rebase --edit-todo
```

示例：你不小心把 `edit` 写成了 `editor`。

## Reword

与其将 `pick` 改为 `edit`，不如改为 `reword`。
    保存并关闭后，Git 会自动重新打开编辑器供你编辑提交信息。
    完成后，流程即告结束——无需执行 `amend` 或 `continue`。
    此方法最适合纯粹修改 **提交信息**。

[amend]:./amend.md
