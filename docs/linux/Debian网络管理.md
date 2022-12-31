# Debian网络管理

---

Debian中存在两套管理网络的方案：

1、/etc/network/interfaces

2、Network-Manager

---

方案1默认存在，当安装DE时，比如Gnome，会产生方案2，两套方案是冲突的。

既能避免方案冲突又能共享配置的解决方案：

1、当Network-Manager发现/etc/network/interfaces被改动的时候，则关闭自己（显示为未托管），除非managed设置成真。

2、当managed设置成真时，/etc/network/interfaces，则不生效。

---

对于安装桌面环境的用户建议

编辑/etc/NetworkManager/NetworkManager.conf

把最后一行的“managed=false”改为“managed=true”

然后重启NetworkManager或系统即可