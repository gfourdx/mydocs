# Mihomo安装笔记

## 一、平台及安装

mihomo支持多种平台, 以PC端举例

1. 先检查CPU指令集

```bash
# 检查是否支持 v3 (需要 AVX2 指令集)
$ grep -q avx2 /proc/cpuinfo && echo True || echo False
# 检查是否支持 v2 (需要 SSE4.2 指令集)
$ grep -q sse4_2 /proc/cpuinfo && echo True || echo False
# 统一检查
$ lscpu | grep -o "avx2" && echo "Supports v3" || (lscpu | grep -o "sse4_2" && echo "Supports v2" || echo "Supports v1")
```

2. 下载对应的版本

```bash
# 获取下载地址(以debian amd64 v3为例)
$ curl -s https://api.github.com/repos/MetaCubeX/mihomo/releases/latest | grep browser_download_url | grep linux-amd64-v3 | grep '\.deb' | cut -d '"' -f 4
```

> 小扩展:
>
> ```bash
> # 此命令可获取文件名称
> $ basename https://github.com/MetaCubeX/mihomo/releases/download/v1.19.16/mihomo-linux-amd64-v3-v1.19.16.deb
> ```
>
> 玩客云是Armv7, 其下载链接是:
>
> ```bash
> % curl -s https://api.github.com/repos/MetaCubeX/mihomo/releases/latest | grep browser_download_url | grep linux | grep armv7 | grep deb | cut -d '"' -f 4
> ```

3. 下载

   ```bash
   $ curl -sL -o mihomo-linux-amd64-v3-v1.19.16.deb https://github.com/MetaCubeX/mihomo/releases/download/v1.19.16/mihomo-linux-amd64-v3-v1.19.16.deb
   ```

4. 卸载旧版本(可选)

   ```bash
   $ sudo dpkg --purge mihomo
   ```

5. 安装新版本

   ```bash
   $ sudo dpkg -i mihomo-linux-amd64-v3-v1.19.16.deb
   ```

## 二、配置

1. 下载订阅文件

   ```bash
   $ sudo curl -sL -o /etc/mihomo/config.yaml https://example.com/xxx
   ```

2. 下载UI

   ```bash
   # 获取下载地址
   $ curl -s https://api.github.com/repos/MetaCubeX/metacubexd/releases/latest | grep browser_download_url | cut -d '"' -f 4
   ```

3. 下载

   ```bash
   $ sudo curl -sL -o /etc/mihomo/metacubexd.tgz https://github.com/MetaCubeX/metacubexd/releases/download/v1.202.0/compressed-dist.tgz
   ```

4. 解压

   ```bash
   $ sudo mkdir /etc/mihomo/metacubexd
   $ sudo tar -xf /etc/mihomo/metacubexd.tgz -C /etc/mihomo/metacubexd
   $ sudo rm /etc/mihomo/metacubexd.tgz
   ```

5. 修改启动配置

   ```bash
   $ sudo systemctl edit mihomo.service
   # 写入:
   [Service]
   ExecStart=
   ExecStart=/usr/bin/mihomo -d /etc/mihomo -ext-ui /etc/mihomo/metacubexd -secret yourpassword
   ```

6. 刷新配置

   ```bash
   sudo systemctl daemon-reload
   ```

7. 自动服务及开机自启

   ```bash
   sudo systemctl start mihomo
   sudo systemctl enable mihomo
   ```

## 三、访问管理页面

http://192.168.x.x:9090/ui/#/

