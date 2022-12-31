# 设置Mac的Hostname

---

## 名词

Mac中与之相关的有3个:

- ComputerName
- LocalHostName
- HostName

## 设置

ComputerName和LocalHostName可以通过`系统偏好设置` -> `分享`设置

HostName只能通过命令设置, 可设置临时生效(重启后失效)或永久生效

临时:

```
hostname MyMBP
```

永久:

```
scutil --set HostName MyMBP
```

## 查看

```
$ sudo scutil --get ComputerName
MyMacBookPro
$ sudo scutil --get LocalHostName
MyMBP
$ sudo scutil --get HostName
MBP
```

