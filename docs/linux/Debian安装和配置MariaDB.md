# Debian安装和配置MariaDB

---

## 安装

```
apt install -y mariadb-server mariadb-client
```

## 配置

### 数据库配置

```
# 执行下方命令配置mysql
mysql_secure_installation

# 执行上方命令出现交互式会话
Enter current password for root (enter for none):    # 此处直接按下回车
Switch to unix_socket authentication [Y/n]           # 输入n后按下回车
Change the root password? [Y/n]                      # 输入y后按下回车
New password:                                        # 设置密码
Re-enter new password:                               # 重复刚刚设置的密码
Remove anonymous users? [Y/n]                        # 输入y后按下回车
Disallow root login remotely? [Y/n]                  # 输入n后按下回车
Remove test database and access to it? [Y/n]         # 输入y后按下回车
Reload privilege tables now? [Y/n]                   # 输入y后按下回车

# 登录mysql继续配置，注意：建议把第三条命令中的"password"改成上面设置的密码，保持一致
mysql -uroot -p
use mysql
grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
flush privileges;
exit
```

### 数据库文件配置

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

找到`bind-address = 127.0.0.1`

改为`bind-address = 0.0.0.0`

### 重启服务

```
service mariadb restart
```

或

```
systemctl restart mariadb
```

