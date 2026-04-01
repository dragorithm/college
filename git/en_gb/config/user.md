# User

## Common Keys

- **name**: The username displayed when committing
- **email**: The email address displayed when committing

These two keys are mandatory, as they identify the author and committer of a commit record.

## Other Keys

Less common in most scenarios, but occasionally useful:

- **useConfigOnly**: If set to `true`, Git will only use user information from configuration files and will not attempt to infer it from the system environment
- **signingKey**: Specifies the GPG key ID used for signing commits

## Tips

- For a single user, **user‑level (global)** configuration is sufficient.
- For multiple users, **repository‑level (local)** configuration can be used to distinguish identities.

Common multi‑user examples:

- Multiple developers sharing the same device, needing to differentiate commit records
- One device used for both work and school projects, avoiding identity confusion
- Contributing to open‑source projects with a specific email address instead of a personal one
