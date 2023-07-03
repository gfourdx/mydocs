# Debian安装和配置MariaDB

---

## 安装

```
apt install -y mariadb-server mariadb-client
```

## 配置

### 修改配置文件监听所有IP

```
sed -i 's@bind-address            = 127.0.0.1@bind-address            = 0.0.0.0@g' /etc/mysql/mariadb.conf.d/50-server.cnf
```

### 启动服务

```
service mariadb start
```

### 安全配置

```
# 执行下方命令配置mysql
mysql_secure_installation

# 执行上方命令出现交互式会话(根据实际要求配置即可)
Enter current password for root (enter for none):    # 此处直接按下回车
Switch to unix_socket authentication [Y/n]           # 输入n后按下回车
Change the root password? [Y/n]                      # 输入y后按下回车
New password:                                        # 设置密码
Re-enter new password:                               # 重复刚刚设置的密码
Remove anonymous users? [Y/n]                        # 输入y后按下回车
Disallow root login remotely? [Y/n]                  # 输入n后按下回车
Remove test database and access to it? [Y/n]         # 输入y后按下回车
Reload privilege tables now? [Y/n]                   # 输入y后按下回车

# 登录mysql继续配置，注意：第三条命令中的"mariadb"是密码，建议与上面的密码保持一致
mysql -u root
use mysql
grant all privileges on *.* to 'root'@'%' identified by 'mariadb' with grant option;
flush privileges;
exit
```

### 新增允许从所有IP登录的root账户并授权访问所有数据库

`mysql -u root`

`use mysql`

`grant all privileges on *.* to 'root'@'%' identified by 'mariadb' with grant option;`

`flush privileges;`

`exit`
