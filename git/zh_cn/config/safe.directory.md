# 安全目录

从 Git **2.35** 版本开始，引入了一种名为 `safe.directory` 的新机制，用于防止在不受信任的仓库中执行 Git 操作（参见 [CVE-2022-24765]）。

[CVE-2022-24765]:https://nvd.nist.gov/vuln/detail/cve-2022-24765

当 Git 检测到潜在不安全的仓库时，可能会报告以下错误：

```txt
fatal: detected dubious ownership in repository
```

这通常发生在以下情况：

- 仓库归属于其他用户
- 仓库位于网络文件系统上（例如 SMB、NAS、WSL 挂载）
- Git 以不同权限执行（例如使用 `sudo`）

## 如何修复

### 将仓库添加到受信任列表

```bash
git config --global --add safe.directory <绝对路径>
```

示例：

```bash
git config --global --add safe.directory /home/dragon/college
```

将路径添加到 `safe.directory` 可绕过 Git 的所有权检查。

请确保该仓库值得信赖，因为这可能会引入安全风险。

### 信任所有仓库

不建议在真正的生产环境中使用

```bash
git config --global --add safe.directory '*'
```

这将完全禁用 Git 的安全检查，因此仅应在受控或隔离的环境中使用。
