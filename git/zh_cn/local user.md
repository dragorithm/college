# 本地用户

## 本地用户配置的应用场景

在使用 Git 时, 若遇到以下场景, 需要为不同仓库配置独立身份

- 多开发者共享同一台设备, 需区分不同开发者的提交记录
- 同一设备区分工作与学校项目, 避免身份混淆
- 参与开源项目时, 使用特定邮箱而非常用邮箱

此时可使用 **Git 本地配置(local)**, 该配置仅对当前仓库生效, 且不会同步到远程仓库

## 命令行配置(推荐方式)

通过命令行可快速配置当前仓库的用户信息:

### 基础语法

```bash
git config --local <section>.<key> <value>

```

*注意*: 在仓库目录中执行时, 默认作用于 local 配置, 可省略 `--local` 参数

### 设置用户名

```bash
git config user.name <name>
```

### 设置邮箱

```bash
git config user.email <email>
```

### 完整示例

```bash
git config user.name "dragon"
git config user.email "dragon@dragon.top"
```

## 直接编辑配置文件

`git config`本质上就是编辑`.git/config`, 可直接编辑该文件完成配置

### 配置文件格式

```ini
[user]
    name = <name>
    email = <email>
```

### 例如

```ini
[user]
    name = dragon
    email = dragon@dragon.top
```

*注意*: `.git/config`文件的`[user]`配置不会被 Git 跟踪, 也不会同步到远程仓库

## 错误配置处理

若误配置了错误字段, 如`git config config.name 'dragon'`可通过以下命令删除:

```bash
git config --unset config.name
```

## 查看当前配置

### 查看所有配置(含文件路径)

```bash
git config --list --show-origin
```

*注意*: `--show-origin`参数可显示配置项来源的文件路径

### 查看指定配置项

```bash
git config --get <section>.<key>
```

### 示例: 查看用户名

```bash
git config --get user.name
```

## 配置优先级说明

Git 配置存在优先级关系(从高到低):

1. 本地配置(local), 仅对当前仓库生效
2. 全局配置(global), 对当前用户所有仓库生效
3. 系统配置(system), 对系统所有用户生效

当存在冲突时, 高优先级配置会覆盖低优先级配置
