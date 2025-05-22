# Linux配置IP和DNS

---

## 一、配置IP

### Debian

#### 编辑配置文件

```
vim /etc/network/interfaces
```

#### 追加配置内容

##### 动态IP

```
auto eth1
iface eth1 inet dhcp
```

##### 静态IP

```
auto eth1
iface eth1 inet static
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.1
# dns-nameservers 223.5.5.5 223.6.6.6  # 此处可配置DNS
# hwaddress 00:22:6D:50:42:A9  # 此处可配置MAC地址
```

重启网络服务

```
# 方法1
/etc/init.d/networking restart
# 方法2
service networking restart
# 方法3
systemctl restart networking
```

#### 配置DNS

编辑配置文件

```
vim /etc/resolv.conf
```

修改配置内容

```
nameserver 223.5.5.5
nameserver 223.6.6.6
```

### CentOS

#### 编辑配置文件

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

#### 修改配置内容

##### 动态IP和DNS

```
BOOTPROTO=dhcp      # 设置为dhcp开启DHCP服务，设置为static启用静态IP地址
MM_Controlled=no    # network mamager的参数，实时生效，不需要重启
ONBOOT=yes          # 设置为yes，开机自动启用网络连接
USERCTL=yes         # 是否允许非root用户控制该设备      
DNS1=223.5.5.5
DNS2=223.6.6.6
```

##### 静态IP和DNS

```
ONBOOT=yes
BOOTPROTO=static
MM_Controlled=no
USERCTL=yes
IPADDR=192.168.254.1
NETMASK=255.255.255.0
GATEWAY=192.168.254.254
DNS1=114.114.114.114
DNS2=8.8.8.8
```

#### 重启网络服务

```
systemctl restart network
```

