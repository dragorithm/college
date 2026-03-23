# git commit

Commit messages must be written in English for consistency and readability.

If the git documentation is updated, commit using the new conventions, including the git documentation itself.

## rule

\[type\](file path 1, file path 2, ..., file path n): message  
description

foot

## description

required

- [type]
- (file path 1, file path 2, ..., file path n)
- : message

## type

- **feat**: Feature
  - add: Introduces a new feature; may involve adding a new module or new content within an existing module
  - del: Removes content from a module or deletes an entire module
  - up: Updates an existing feature
- **fix**: Defect fix
  - security: Fixes security‑related issues
  - defect: Fixes non‑security defects
- **docs**: Update documentation under ./docs and the root `README.md`
  - git: Update Git-related content
  - '': Other; no sub-type required
- **style**: Format adjustments do not affect the logic
- **refactor**: Rewrite the code No new features No bug fixes
- **perf**: Performance optimization
- **revert**: Rollback commit
- **chore**: other

## Three‑Layer Commit Structure

- type: Describes what was done.
- scope: Describes where the change was made.
- message: Should focus on why the change was made; add what-details only if necessary.
    When submitting messages, all text should be in lowercase, words should be separated by spaces, and no full stop should be added at the end.
