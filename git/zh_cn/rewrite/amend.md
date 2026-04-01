# 重写最近的提交

[index](./index.md)

```bash
git commit --amend
```

此命令通常用于：

- 修改提交信息
- 添加在 `git add` 过程中遗漏的文件

*注意*：在执行此命令之前，必须确认 **暂存区** 的内容，以免意外提交本不应添加的文件。

## 示例

编辑提交：

```bash
git commit --amend
```

编辑器将打开类似的界面：

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

大多数以 `#` 开头的行可以忽略。
    请关注没有 `#` 的行，因为那里是 **提交信息**。
    例如，你可以将其修改为：

```txt
[feat.del](notes): remove duplicate notes with vscode
```

然后保存并关闭编辑器。

如您所见，**提交信息**已更新，**哈希值**也发生了变化。

```bash
commit 18a2c9e9510a2872dbfa5ef3fec51695a4d905f3
Author: gzs <dragon@dragorithm.top>
Date:   Wed Apr 1 11:36:38 2026 +0800

    [feat.del](notes): remove duplicate notes with vscode
```
