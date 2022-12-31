# Windows 11 安装配置 MySQL 5.7.38

---

1、下载

https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.38-winx64.zip

2、创建目录

```
mkdir C:\MySQL\5.7.38
```

3、配置环境变量

```
setx path "%path%;C:\MySQL\5.7.38\bin"
```

4、解压`mysql-5.7.38-winx64.zip`到目录`C:\MySQL\5.7.38`

5、在目录`C:\MySQL\5.7.38 `中创建配置文件`my.ini`并写入如下内容

```
[mysqld]
# set basedir to your installation path
basedir=C:\\MySQL\\5.7.38
# set datadir to the location of your data directory
datadir=C:\\MySQL\\5.7.38\\data
```

6、下载vcredist_x64.exe并安装

[Visual C++ Redistributable Packages for Visual Studio 2013](https://www.microsoft.com/zh-CN/download/details.aspx?id=40784)

7、初始化

```
mysqld --defaults-file=C:\MySQL\5.7.38\my.ini --initialize --console
```

最后一行末尾部分输出的是root用户的临时密码, 类似`98h/D+zd#WB&`, 记录下来

8、安装服务并启动

```
mysqld --install
sc start mysql
```

9、配置数据库

```
mysql_secure_installation
```

10、设置root用户可远程登录

```
mysql -u root -p
grant all privileges on *.* to 'root'@'%' identified by 'mysql' with grant option;
flush privileges;
exit
```

搞定!