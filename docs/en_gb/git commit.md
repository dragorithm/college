# git commit

Commit messages must be written in English for consistency and readability.

If the git documentation is updated, commit using the new conventions, including the git documentation itself.

## rule

\[type\](scope): message  
description

foot

## description

required

- [type]
- (scope)
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

## scope

- **college**: Content related to academy assignments
- **docs**: Content related to the docs directory or the `README.md`.
    This may be ignored when the type is docs.
- **git**: Content related to Git
- **notes**: Content related to study notes
- **vscode**: Content related to VS Code

## Core Three‑Layer Commit Structure

- type: Describes what was done.
- scope: Describes where the change was made.
- message: Describes what was done.

*Note*: Commit messages should be limited to a single sentence, all text should be written in lowercase, with words separated by spaces, and without a full stop at the end.
    An exception is that proper nouns may use capital letters, such as OpenZFS, Toothless, and Darkstalker.

## Extended Commit Structure

- description: Filled in only when necessary to explain why the change was made.

*Note*: In most cases, the reasoning behind a change should be documented elsewhere rather than inside the commit message.
