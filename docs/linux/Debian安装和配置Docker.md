# Debian安装和配置Docker

---

## 在线安装

```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

或

```
wget -qO- https://get.docker.com | sh
```

## 镜像加速和配置IP地址段

```
mkdir -p /etc/docker

tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://t6wp3k6n.mirror.aliyuncs.com"],
  "bip": "192.168.255.1/24"
}
```

> registry-mirrors中的加速地址从阿里云获取，获取地址：
>
> https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
>
> bip是修改Docker默认分配bridge网络的地址段
>
> 若要配置DNS, 请添加 "dns": ["223.5.5.5", "223.6.6.6"]
>
> 若要配置Docker镜像和容器的存储配置, 请添加 "data-root": "/path/to/custom"

## 非root用户取消sudo限制

### 启动docker

```
service docker start
```

或

```
systemctl docker start
```

### 将当前用户加入docker组

```
sudo gpasswd -a ${USER} docker
```

### 刷新group缓存

```
newgrp - docker 
```

---

## 离线安装

### 访问官网下载地址

```
https://download.docker.com/linux/debian/dists/bullseye/pool/stable/amd64/
```

> 该网站 https://download.docker.com/ 可离线下载linux、mach和windos的离线安装包
>
> 对于linx，可选择支持的发行版、版本和架构等
>
> 本案例是Debian bullseye(11)的amd64架构

### 下载离线安装包

```
wget https://download.docker.com/linux/debian/dists/bullseye/pool/stable/amd64/containerd.io_1.4.6-1_amd64.deb

wget https://download.docker.com/linux/debian/dists/bullseye/pool/stable/amd64/docker-ce-cli_20.10.7~3-0~debian-bullseye_amd64.deb

wget https://download.docker.com/linux/debian/dists/bullseye/pool/stable/amd64/docker-ce_20.10.7~3-0~debian-bullseye_amd64.deb

wget https://download.docker.com/linux/debian/dists/bullseye/pool/stable/amd64/docker-ce-rootless-extras_20.10.7~3-0~debian-bullseye_amd64.deb

wget https://download.docker.com/linux/debian/dists/bullseye/pool/stable/amd64/docker-scan-plugin_0.8.0~debian-bullseye_amd64.deb
```

### 离线安装

```
dpkg -i containerd.io_1.4.6-1_amd64.deb

dpkg -i docker-ce-cli_20.10.7~3-0~debian-bullseye_amd64.deb

dpkg -i docker-ce_20.10.7~3-0~debian-bullseye_amd64.deb

dpkg -i docker-ce-rootless-extras_20.10.7~3-0~debian-bullseye_amd64.deb

dpkg -i docker-scan-plugin_0.8.0~debian-bullseye_amd64.deb
```

