# Editor

When Git requires you to edit a file, for example with `git commit --amend`, it will open an editor.
    However, this editor is not always the one you are accustomed to.

Git supports changing the editor via **environment variable**s or **Git configuration**.

- **Git configuration**: Change core.editor, see [config index](./config/index.md)
- **Environment variable**: Change `GIT_EDITOR`

## Priority

Git searches for the editor in the following order:

1. Environment variable `GET_EDITOR`
2. Git configuration option `core.editor`
3. Environment variable `VISUAL`
4. Environment variable `EDITOR`
5. System default editor(usually `vi`)

## Values

### Blocking

Common editors can be launched simply by specifying their name, provided the name exists in the `PATH` environment variable.

Examples:

- vi
- vim
- notepad

### Non‑Blocking

Some modern editors(such as VS Code) are non-blocking by default.
    Git will continue execution immediately after opening the editor.

Therefore, you need to add the `--wait` parameter so that Git waits until the editor is closed before continuing:

- `code --wait`

### Full Path

For editors not available in the environment variables, you can specify the full path directly:

- `C:\WINDOWS\System32\notepad.exe`
- `/usr/bin/vim`

## Examples

Configure VS Code as the editor via Git Configuration:

```bash
git config --global core.editor "code --wait"
```

Configure vim as the editor via environment variable:

```bash
export GIT_EDITOR='vim'
```
