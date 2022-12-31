# WSL使用Debian启动Docker报错

---

针对如下错误提示的解决办法：

**`grep: /etc/fstab: No such file or directory`**

参考：https://github.com/microsoft/WSL/discussions/4872

执行以下命令解决

```
sudo touch /etc/fstab

sudo update-alternatives --set iptables /usr/sbin/iptables-legacy

sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

sudo service docker start
```

