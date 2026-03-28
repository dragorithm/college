# Git 配置

Git 配置定义了用户在使用 Git 时的偏好设置，包括用户名、电子邮件地址、编辑器等。
    通过调整三个选项，用户可以定制 Git 以更好地适应其工作流程，从而提高版本控制的效率和一致性。

Git 配置分为三个级别：

- **系统级别**：适用于所有用户
- **用户级别**：适用于当前用户
- **仓库级别**：仅适用于当前仓库

Git 配置的优先级（从高到低）依次为：

1. 仓库级别
2. 用户级别
3. 系统级别

*注意*：无论处于哪个级别，对 Git 配置的更改都不会同步到远程仓库。

Git 配置可通过以下两种方式修改：

- 命令行（推荐）
- 配置文件

*注意*：修改系统级配置可能需要管理员权限。

## 通过命令行查询

### 列出所有配置项

```bash
git config --show-origin --list
```

`--show-origin` 会显示每个条目来自哪个配置文件。
    该选项是可选的，但建议使用以获得更详细的输出。

### 查询特定配置项

```bash
git config --show-origin --get <section>. <key>
```

此命令显示当前生效的值。

### 查询配置项的定义位置

```bash
git config --show-origin --get-regexp <section>.<key>
```

此命令按系统级、用户级和仓库级顺序输出定义结果，并跳过未定义该键的级别。

## 通过命令行删除配置

有时您可能会输入错误的配置，或者想要删除未使用的条目。
    在这种情况下，您可以通过命令行删除配置项。

### 基本语法

```bash
git config [--system|--global|--local] --unset <section>.<key>
```

### 删除多个值

如果同一键存在多个值，请使用：

```bash
git config [--system|--global|--local] --unset-all <section>.<key>
```

否则，Git 会发出警告：`warning: <section>.<key> has multiple values`

### 示例

删除用户级编辑器：

```bash
git config --global --unset core.editor
```

删除错误添加的条目：

```bash
git config --unset abcd.efg
```

删除所有安全目录：

```bash
git config --global --unset-all safe.directory
```

## 配置文件

通过命令行进行的更改最终会修改配置文件。
    这些文件是采用 **INI** 格式的纯文本文件，也可以手动编辑。

### 基本格式

```ini
[section]
    key = value
```

### 位置

- 系统级别：`/etc/gitconfig` 或 `C:\ProgramData\Git\config`
- 用户级别：`~/.gitconfig`
- 仓库级别：`./.git/config`

### 示例

设置编辑器：

```ini
[core]
    editor = vi
```
