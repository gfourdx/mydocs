# Mysql时区

---

查看时区：
**`show variables like '%time_zone%';`**

结果显示为：

| Variable_name    | Value  |
| ---------------- | ------ |
| system_time_zone | UTC    |
| time_zone        | SYSTEM |

含义解释：

system_time_zone: 系统时区。当服务器启动时，它试图自动确定主机的时区，并使用它来设置system_time_zone系统变量。此后，该值不会改变

time_zone: 当前的时区，这个变量用来为每个连接的客户端初始化时区。默认情况下，它的初始值是'SYSTEM'（这意味着，"使用system_time_zone的值"）

---

修改时区：

一共有三种方式设置时区：

1、运行期间

**`set global time_zone='+8:00';`**

或者

**`set global time_zone='Asia/Shanghai';`**

> 1、**注意修改时区后，需要断开与数据库的连接，再重新连接，否则此时查看时区发现没变，但实际上设置成功！**

> 2、临时修改命令是 **`set time_zone='Asia/Shanghai;'**，退出mysql重新进去会发现失效

2、启动命令

**`--default-time-zone=timezone`**

3、配置文件

**`default-time-zone=timezone`**

