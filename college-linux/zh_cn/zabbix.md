# zabbix

本文档记录课程学习环境的部署过程，不包含任何实际生产环境的相关信息。

消除干扰

```bash
ufw disable
systemctl stop apparmor
systemctl disable apparmor
hostnamectl set-hostname dragon
```

更新系统

```bash
apt update
apt upgrade
```

安装数据库系统和nginx

```bash
apt install mariadb-server nginx
```

安装Zabbix软件源

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.4+ubuntu24.04_all.deb
apt update
```

安装Zabbix服务器及前端

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts
```

创建初始数据库

```bash
mariadb
```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by ‘password’;
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

在Zabbix服务器主机导入初始架构和数据。

```bash
# 系统将提示输入新创建的密码
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mariadb --default-character-set=utf8mb4 -u zabbix -p zabbix
```

导入数据库架构后禁用 log_bin_trust_function_creators 选项。

```bash
mariadb
```

```sql
set global log_bin_trust_function_creators = 0;
quit;
```

配置Zabbix服务器的数据库

```bash
sed -i ‘s/# DBPassword=/DBPassword=password/’ /etc/zabbix/zabbix_server.conf
```

配置nginx配置文件

```bash
rm /etc/nginx/sites-enabled/default
```

启动Zabbix服务器进程

```bash
# 启动Zabbix服务器进程
systemctl restart zabbix-server nginx php8.3-fpm

# 并设置为系统启动时自动运行
systemctl enable zabbix-server nginx php8.3-fpm
```

选择 `zabbix-server` 监听端口（默认 10051）

```bash
systemctl status zabbix-server | grep “Main PID:”
# 示例：Main PID: 1198 (zabbix_server)

ss -tulnp | grep 1198
```

安装 Zabbix 代理 2

```bash
apt install zabbix-agent2
```

配置 zabbix_agent2 属性

```bash
hostname
# dragon

# 确保与 `hostname` 匹配
tee /etc/zabbix/zabbix_agent2.d/dragon.conf > /dev/null <<EOF
Hostname=dragon
EOF

systemctl restart zabbix-agent2
```

更新语言环境

```bash
sed -i ‘s/# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/’ /etc/locale.gen
locale-gen
systemctl restart php8.3-fpm
```

强制刷新数据

```bash
zabbix_server -R config_cache_reload
```
