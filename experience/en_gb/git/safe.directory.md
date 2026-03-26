# Safe Directory

Starting from Git **2.35**, a new mechanism called `safe.directory` was introduced to prevent Git operations from being executed inside untrusted repositories(see [CVE-2022-24765]).

[CVE-2022-24765]:https://nvd.nist.gov/vuln/detail/cve-2022-24765

Git may report the following error when it detects a potentially unsafe repository:

```txt
fatal: detected dubious ownership in repository
```

This typically happens when:

- The repository is owned by a different user
- The repository is located on network filesystem(e.g., SMB, NAS, WSL mount)
- Git is executed with different privileges(e.g., using `sudo`)

## How to fix

### Add the repository to the trusted list

```bash
git config --global --add safe.directory <absolute-path>
```

Example:

```bash
git config --global --add safe.directory /home/dragon/college
```

Adding a path to `safe.directory` bypasses Git's ownership checks.

Make sure the repository is trustworthy, as this may introduce security risks.

### Trust all repositories

Not recommended for real production environments

```bash
git config --global --add safe.directory '*'
```

This disables Git's safety checks entirely, so use it only in controlled or isolated environments.
