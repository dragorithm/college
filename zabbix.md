# zabbix & htop

This document records the deployment of the learning environment for coursework and does not contain any information pertaining to actual production environments.

Eliminate interference

```bash
ufw disable
systemctl stop apparmor
systemctl disable apparmor
hostnamectl set-hostname dragon
```

Update system

```bash
apt update
apt upgrade
```

Install database system and nginx

```bash
apt install mariadb-server nginx
```

Install Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.4+ubuntu24.04_all.deb
apt update
```

Install Zabbix server, frontend

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts
```

Create initial database

```bash
mariadb
```

```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

On Zabbix server host import initial schema and data.

```bash
# You will be prompted to enter your newly created password.
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mariadb --default-character-set=utf8mb4 -uzabbix -p zabbix
```

Disable log_bin_trust_function_creators option after importing database schema.

```bash
mariadb
```

```sql
set global log_bin_trust_function_creators = 0;
quit;
```

Configure the database for Zabbix server

```bash
sed -i 's/# DBPassword=/DBPassword=password/' /etc/zabbix/zabbix_server.conf
```

Configure the nginx conf file

```bash
rm /etc/nginx/sites-enabled/default
```

Start Zabbix server processes

```bash
# Start Zabbix server processes
systemctl restart zabbix-server nginx php8.3-fpm

# And make it start at system boot.
systemctl enable zabbix-server nginx php8.3-fpm
```

Select `zabbix-server` listen port(default 10051)

```bash
systemctl status zabbix-server | grep "Main PID:"
# example: Main PID: 1198 (zabbix_server)

ss -tulnp | grep 1198
```

Install Zabbix agent 2

```bash
apt install zabbix-agent2
```

Configure zabbix_agent2 properties

```bash
hostname
# dragon

# make sure equal `hostname`
tee /etc/zabbix/zabbix_agent2.d/dragon.conf > /dev/null <<EOF
Hostname=dragon
EOF

systemctl restart zabbix-agent2
```

Update Language

```bash
sed -i 's/# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen
locale-gen
systemctl restart php8.3-fpm
```

Force data refresh

```bash
zabbix_server -R config_cache_reload
```
