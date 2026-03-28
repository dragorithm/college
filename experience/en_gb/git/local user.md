# Local User

## When to Use Local User Configuration

In Git, you many need different identities for different repositories in situations such as:

- Multiple developers sharing the same machine and needing separate commit identities
- Separating work, personal, and school projects avoid mixing commit authors
- Using a specific email address for open-source contributions instead of your primary one

In the cases, **Git local configuration** is idel. It applies **only to the current repository** and is **never pushed to the remote.**

## Command-Line Configuration(Recommended)

You can quickly configure user information for the current repository using the command line..

### Basic Syntax

```bash
git config --local <section>.<key> <value>
```

*Note*: When running inside a repository, Git defaults to **local** scope, so `--local` can be omitted.

### Set Username

```bash
git config user.name <name>
```

### Set Email

```bash
git config user.email <email>
```

### Example

```bash
git config user.name dragon
git config user.email dragon@dragon.top
```

## Editing the Configuration File Directly

`git config`simply modifies `.git/config`.

You may edit this file manually if preferred.

### File Format

```ini
[user]
    name = <name>
    email = <email>
```

### Example

```ini
[user]
    name = dragon
    email = dragon@dragon.top
```

*Note*: The `[user]` section is `.git/config` is **not tracked by Git** and **not include in pushes**.

## Fixing Incorrect Configuration

If you accidentally set the wrong field, such as:

```bash
git config config.name dragon
```
