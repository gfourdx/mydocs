# 安装mysqlclient和psycopg2报错

---

***注：仅适用于Debian系列发行版***

安装mysqlclient报错请安装依赖

```
apt install -y libmariadb-dev
```

> 如果没有依赖libmariadb-dev则安装libmysqlclient-dev

安装psycopg2报错请安装依赖

```
apt install -y libpq-dev
```

如果依旧报错，还需要安装以下依赖

```
apt-get install -y python3-dev build-essential
```
