# 挂载ISO镜像做本地源

---

## 创建挂载目录

```
mkdir /mnt/iso
```



## 挂载ISO

```
mount /dev/cdrom /mnt/iso
```



> 根据实际情况修改 /dev/cdrom

## 备份原文件

### CentOS

```
mv /etc/yum.repos.d /etc/yum.repos.d.bak
```



### Debian

```
mv /etc/apt/sources.list /etc/apt/sources.list.bak
```



## 编辑源文件

### CentOS

```
vim /etc/yum.repos.d/iso.repo
```



### Debian

```
vim /etc/apt/sources.list
```



## 写入以下内容

### CentOS

```
[local]
name=CentOS_ISO
baseurl=file:///mnt/iso
enabled=1
gpgkey=file://mnt/iso/RPM-GPG-KEY-CentOS-7
```

### Debian

```
deb file:///mnt/iso bullseye main contrib non-free
```

> 根据实际情况修改 bullseye 为正确的发行版名称