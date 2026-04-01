# Git Configuration

Git configuration defines user preferences when working with Git, including username, email address, editor, and more.
    By adjusting three options, users can tailor Git to better fit their workflow, improving efficiency and consistency in version control.

Git configurations is divided into three levels:

- **System level**: Applies to all users
- **User level**: Applies to the current user
- **Repository level**: Applies only to the current repository

The priority of Git configuration, from highest to lowest, is:

1. Repository level
2. User level
3. System level

*Note*: Regardless of the level, changes to Git configuration are not synchronised to remote repositories.

Git configuration can be modified in two ways:

- Command line(recommended)
- Configuration files

*Note*: Modifying system-level configuration may require administrator privileges.

## Write Operations via Command Line

### Basic Syntax

```bash
git config [--system|--global|--local] <section>.<key> <value>
```

Scopes:

- System level: `--system`
- User level: `--global`
- Repository: `--local`

Default scope: repository level.

If the configuration does not exists, it will be created; if it already exist, it will overwritten.

Example:

```bash
git config --global core.editor vi
```

### Append Syntax

```bash
git config [--system|--global|--local] --add <section>.<key> <value>
```

use this when you need to assign multiple values to the same key.

## Querying via Command Line

### List All Configuration

```bash
git config --show-origin --list
```

`--show-origin`displays which configuration file each entry comes from.
    It is optional but recommended for more informative output.

### Query a Specific Configuration Item

```bash
git config --show-origin --get <section>.<key>
```

This command shows the value currently in effect.

### Query Where a Configuration Item Is Defined

```bash
git config --show-origin --get-regexp <section>.<key>
```

This outputs system-level, user-level, and repository-level definitions in order, skipping levels where the key is not defined.

## Deleting Configuration via Command Line

Sometimes you may enter incorrect configuration or want to remove unused entries.
    In such cases, you can delete configuration items via command line.

### Basic Syntax

```bash
git config [--system|--global|--local] --unset <section>.<key>
```

### Deleting Multiple Values

If multiple values exits ofr the same key, use:

```bash
git config [--system|--global|--local] --unset-all <section>.<key>
```

Otherwise, Git will warn: `warning: <section>.<key> has multiple values`

### Examples

Delete the user-level editor:

```bash
git config --global --unset core.editor
```

Delete an incorrectly added entry:

```bash
git config --unset abcd.efg
```

Delete all safe directories:

```bash
git config --global --unset-all safe.directory
```

## Configuration Files

Changes made via command lien ultimately modify configuration files.
    These files are plain text in **INI** format and can also be edited manually.

### Basic Format

```ini
[section]
    key = value
```

### Locations

- System level: `/etc/gitconfig` or `C:\ProgramData\Git\config`
- User level: `~/.gitconfig`
- Repository level: `./.git/config`

### Example

Set the editor:

```ini
[core]
    editor = vi
```
