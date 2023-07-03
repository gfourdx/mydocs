# Debian安装和配置PostgreSQL

---

## 安装

参考：[PostgreSQL: Linux downloads (Debian)](https://www.postgresql.org/download/linux/debian/)

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
```

## 配置

### 允许任何IP访问

##### Debian11

```
sed -i "s@host    all             all             127.0.0.1/32            md5@host    all             all             0.0.0.0/0            md5@g" /etc/postgresql/13/main/pg_hba.conf
```

##### Debian12

```
sed -i "s@host    all             all             127.0.0.1/32            scram-sha-256@host    all             all             0.0.0.0/0               scram-sha-256@g" /etc/postgresql/15/main/pg_hba.conf
```

### 监听任何IP

##### Debian11

```
sed -i "s@#listen_addresses = 'localhost'@listen_addresses = '*'@g" /etc/postgresql/13/main/postgresql.conf
```

##### Debian12

```
sed -i "s@#listen_addresses = 'localhost'		# what IP address(es) to listen on;@listen_addresses = '*'			# what IP address(es) to listen on;@g" /etc/postgresql/15/main/postgresql.conf
```

### 启动

```
service postgresql start
```

### 切换至Linux用户postgres

```
su - postgres
```

> 一般无需密码即可切换到postgres用户
>
> 如提示输入密码或想设置密码可执行命令 `passwd postgres`

### 修改数据库用户postgres密码

```
psql
alter user postgres with password 'postgres';
\q
exit
```
