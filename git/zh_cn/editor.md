# 编辑器

当 Git 要求你编辑文件时（例如使用 `git commit --amend`），它会打开一个编辑器。
    不过，这个编辑器未必是你习惯使用的那个。

Git 支持通过 **环境变量** 或 **Git 配置** 来更改编辑器。

- **Git 配置**：修改 `core.editor`，参见 [config index](./config/index.md)
- **环境变量**：修改 `GIT_EDITOR`

## 优先级

Git 按以下顺序搜索编辑器：

1. 环境变量 `GIT_EDITOR`
2. Git 配置选项 `core.editor`
3. 环境变量 `VISUAL`
4. 环境变量 `EDITOR`
5. 系统缺省编辑器（通常为 `vi`）

## 值

### 阻塞模式

只要名称存在于 `PATH` 环境变量中，只需指定其名称即可启动常见编辑器。

示例：

- vi
- vim
- notepad

### 非阻塞模式

某些现代编辑器（如 VS Code）缺省采用非阻塞模式。
    Git 会在打开编辑器后立即继续执行。

因此，您需要添加 `--wait` 参数，以便 Git 等待编辑器关闭后再继续：

- `code --wait`

### 完整路径

对于环境变量中未包含的编辑器，您可以直接指定完整路径：

- `C:\WINDOWS\System32\notepad.exe`
- `/usr/bin/vim`

## 示例

通过 Git 配置将 VS Code 设为编辑器：

```bash
git config --global core.editor “code --wait”
```

通过环境变量将 vim 配置为编辑器：

```bash
export GIT_EDITOR=‘vim’
```
