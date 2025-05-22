# Linux中环境GO的环境变量

---

### 使用 Linux 命令配置

以GOPATH举例

方式1

```
~# echo $GOPATH               # 未配置环境

~# go env|grep GOPATH         # 查看GOPATH, 是默认值
GOPATH="/root/go"
~# 
~# export GOPATH=/root/gos    # 配置环境变量
~# 
~# echo $GOPATH               # 查看环境变量, 已改变
/root/gos
~# 
~# go env|grep GOPATH         # 查看GOPATH, 已改变
GOPATH="/root/gos"
~# 
```

结论: **export GOPATH=/root/gos 可更改go env**

注意: 

1、使用export方式设置的环境变量, 仅对当前终端起作用, 关闭当前终端环境变量失效, 不关闭当前终端也不会影响其他新开终端的环境变量;

2、想要永久生效请将命令`export GOPATH=/root/gos`写入 .bashrc 或 .profile中

3、如果想要取消该环境变量, 执行命令`unset GOPATH` , 或者从  .bashrc 或 .profile中删除`export GOPATH=/root/gos`后再执行source命令

---

### 使用 go env 命令配置

```
~# go env|grep GOPATH            # 查看GOPATH, 是默认值
GOPATH="/root/go"
~# 
~# go env -w GOPATH=/root/gos    # 配置环境变量
~# 
~# cat ~/.config/go/env          # 环境变量已写入配置文件
GOPATH=/root/gos
~# 
~# go env|grep GOPATH            # 查看GOPATH, 已改变
GOPATH="/root/gos"
~# 
~# go env -u GOPATH              # 取消配置环境变量
~# 
~# cat ~/.config/go/env          # 环境变量已从配置文件中删除
~# 
~# go env|grep GOPATH            # 查看GOPATH, 已变成默认值
GOPATH="/root/go"
~# 
~# export GOPATH=/root/gos
~# go env -w GOPATH=/root/go 
warning: go env -w GOPATH=... does not override conflicting OS environment variable
~# 
```

结论: 

1、此`go env -w`的方式配置环境变量, 会写入配置文件并永久生效, 想要撤销需要执行命令`go env -u`

2、`go env -w`的方式和通过命令`export`永久生效的效果一样, 区别就是写入的文件(前者是~/.config/go/env, 后者是~/.bashrc或~/.profile)和管理方式(前者是go管理,后者是linux管理)

3、`go env -w`的优先级比`export`低, 如果先用`export`配置了环境变量, `go env -w`不会生效