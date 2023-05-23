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

```
sed -i "s@host    all             all             127.0.0.1/32            md5@host    all             all             0.0.0.0/0            md5@g" /etc/postgresql/13/main/pg_hba.conf
```

### 监听任何IP

```
sed -i "s@#listen_addresses = 'localhost'@listen_addresses = '*'@g" /etc/postgresql/13/main/postgresql.conf
```

### 修改Linux用户postgres密码

```
passwd postgres
```

输入两次密码确认即可

### 启动

```
service postgresql start
```

### 切换至Linux用户postgres

```
su - postgres
```

输入上面设置的密码回车即可

### 修改数据库用户postgres密码

```
psql
alter user postgres with password 'postgres';
\q
exit
```

> 上述第2条命令中最后的'postgres'即是密码，根据实际情况设置即可

### 重启服务

```
service postgresql restart
```
或
```
systemctl restart postgresql
```
