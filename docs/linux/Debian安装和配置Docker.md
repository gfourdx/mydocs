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

## 配置IP地址段和代理

```
mkdir -p /etc/docker

tee /etc/docker/daemon.json <<-'EOF'
{
  "bip": "192.168.255.1/24",
  "proxies": {
    "http-proxy": "172.21.80.1:7890",
    "https-proxy": "http://172.21.80.1:7890",
    "no-proxy": "127.0.0.0/8"
  }
}
```

> bip是修改Docker默认分配bridge网络的地址段
>
> 若要配置DNS, 请添加 "dns": ["223.5.5.5", "223.6.6.6"]
>
> 若要配置Docker镜像和容器的存储配置, 请添加 "data-root": "/path/to/custom"

## 免sudo

```
sudo groupadd docker  # 可以省略
sudo usermod -aG docker $USER
newgrp docker
```

---

## 离线安装

```
https://download.docker.com
```

以Debian12为例:

```
https://download.docker.com/linux/debian/dists/bookworm/pool/stable/amd64/
```

必选

```
containerd
docker-ce-cli
docker-ce
```

可选

```
docker-buildx-plugin
docker-ce-rootless-extras
docker-compose-plugin
docker-scan-plugin
```



