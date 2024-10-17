# 使用lima实现docker

## 下载lima

https://github.com/lima-vm/lima

## 启动docker虚拟机

```
limactl start --name=default --cpus=2 --memory=4 --vm-type=vz --network=vzNAT --mount-type=virtiofs template://docker
```

> 关于各项参数说明请参考: 
>
> https://lima-vm.io/docs/config/vmtype/
>
> https://lima-vm.io/docs/config/network/

## 配置docker命令

### 二进制方式(不建议)

下载二进制文件, 授予执行权限并加入环境变量, 下载地址为: 

https://download.docker.com/mac/static/stable/aarch64/

此时还需配置docker上下文:

```
# 注意将第一条命令中的whls替换成你的用户名
docker context create lima-default --docker "host=unix:///Users/whls/.lima/default/sock/docker.sock"
docker context use lima-default
```

### 别名方式(推荐)

将以下命令添加环境变量:

```
alias docker="lima docker"
```

## 警告处理



```
# 执行命令docker info提示:
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled

# 进入虚拟机
lima

# 配置iptables和ip6tables
echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.conf

# 使配置生效
sudo sysctl -p

# 如果提示:
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables: No such file or directory
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-ip6tables: No such file or directory

# 则需要加载 br_netfilter 模块:
sudo modprobe br_netfilter

# 验证是否加载 br_netfilter 模块成功:
lsmod | grep br_netfilter
# 如果输出中包含 br_netfilter，则说明模块已成功加载

# 配置永久加载 br_netfilter 模块:
echo "br_netfilter" | sudo tee -a /etc/modules

# 此时再执行
sudo sysctl -p

# 最后, 退出虚拟机并重启
limactl stop
limactl start
```

