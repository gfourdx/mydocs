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
sed -i '0,/127.0.0.1\/32/s//0.0.0.0\/0/' /etc/postgresql/15/main/pg_hba.conf
```

### 监听任何IP

```
sed -i "0,/#listen_addresses = 'localhost'/s//listen_addresses = '*'/" /etc/postgresql/15/main/postgresql.conf
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
