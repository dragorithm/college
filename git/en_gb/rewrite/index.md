# Rewriting Commits

The following are common operations in Git that involve rewriting history.

## Notes

Rewriting history will cause commit hashes(SHA-1/ SHA-256) to change.

If the history has already been pushed to a remote branch, rewriting requires using the following command:

```bash
git push --force
```

Or, more safely:

```bash
git push --force-with-lease
```

**Caution**: Forcing a push can be problematic.
    If another developer has pushed new commits which you are overwriting history without pulling first,
    your force push may overwrite the more recent history on the remote branch.

## Contents

- [Rewrite the Most Recent Commit](./amend.md)
- [Rebase](./rebase.md)
