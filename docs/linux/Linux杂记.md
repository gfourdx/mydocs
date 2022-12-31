# Linux杂记

---

## 查看网卡型号：

```
lspci -vv | grep Network
```

## Debian配置区域

```
dpkg-reconfigure locales
```

## 配置中国时区

```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 下载RMP安装包到指定目录并强制安装（忽略依赖关系）

```
dnf install -y --downloadonly --destdir /src/rpm python3 nginx
rpm -ivh --nodeps --force /src/rpm/*.rpm
```

