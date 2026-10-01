# Debian安装和配置Mihomo

以下是 Oracle Cloud 配置 Mihomo 记录

## 一、下载与安装

AMD64

```shell
curl -sL -o mihomo.deb $(curl -s https://api.github.com/repos/MetaCubeX/mihomo/releases/latest | grep browser_download_url | grep linux-amd64-v3 | grep '\.deb' | cut -d '"' -f 4)
```

ARM64

```shell
curl -sL -o mihomo.deb $(curl -s https://api.github.com/repos/MetaCubeX/mihomo/releases/latest | grep browser_download_url | grep linux-arm64 | grep '\.deb' | cut -d '"' -f 4)
```

安装

```shell
sudo dpkg -i mihomo.deb
```

## 二、服务端准备

```shell
echo "UUID: $(cat /proc/sys/kernel/random/uuid)" > proxy.info
mihomo generate reality-keypair >> proxy.info
echo "ShortID: $(openssl rand -hex 8)" >> proxy.info
echo "Domain: www.microsoft.com" >> proxy.info
```

然后

```shell
cat proxy.info
```

注意

`UUID` 、`PrivateKey`、`ShortID`、`Domain`

## 三、服务端配置

```shell
sudo vim /etc/mihomo/config.yaml
```

写入:

```yaml
mode: rule
log-level: info
ipv6: true

listeners:
  - name: vless-reality-in
    type: vless
    listen: 0.0.0.0
    port: 443
    users:
      - username: mihomo
        uuid: "Your UUID"
        flow: xtls-rprx-vision
    reality-config:
      dest: "Your Domain:443"
      private-key: "Your PrivateKey"
      short-id:
        - "Your ShortID"
      server-names:
        - "Your Domain"

rules:
  - MATCH,DIRECT

```

设置权限

```shell
sudo chmod 600 /etc/mihomo/config.yaml
```

测试配置

```shell
sudo mihomo -t -d /etc/mihomo
```

配置自启

```shell
sudo systemctl enable --now mihomo
```

查看状态

```shell
sudo systemctl status mihomo --no-pager
```

检查端口

```shell
sudo ss -lntp | grep ':443'
```

开放IPTables

```shell
sudo iptables -S INPUT
sudo iptables -I INPUT 5 -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -S INPUT
```

持久化IPTables

```shell
# 安装
apt-get install -y --no-install-recommends --no-install-suggests iptables-persistent
# 持久化
sudo netfilter-persistent save
# 检查
sudo grep -- '--dport 443' /etc/iptables/rules.v4
```

## 四、客户端配置

```yaml
mixed-port: 7890
allow-lan: false
bind-address: "*"
ipv6: false
mode: rule
log-level: info
external-controller: 127.0.0.1:9090
secret: ""

dns:
  enable: true
  use-hosts: true
  use-system-hosts: true
  respect-rules: true
  listen: 127.0.0.1:1053
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - "*.lan"
    - "+.local"
    - localhost.ptlogin2.qq.com
  default-nameserver:
    - tls://1.12.12.12
    - 223.5.5.5
    - 119.29.29.29
  nameserver-policy:
    "geosite:cn,private":
      - https://223.5.5.5/dns-query
      - https://223.6.6.6/dns-query
  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  proxy-server-nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  fallback:
    - tls://1dot1dot1dot1.cloudflare-dns.com
    - https://1.0.0.1/dns-query
    - https://1.1.1.1/dns-query
  fallback-filter:
    geoip: true
    geoip-code: CN
    geosite:
      - gfw
    ipcidr:
      - 240.0.0.0/4
      - 0.0.0.0/32
      - 127.0.0.1/32
    domain:
      - "+.google.com"
      - "+.facebook.com"
      - "+.youtube.com"

proxies:
  - name: Oracle-Reality
    type: vless
    server: "Your Server IP"
    port: 443
    uuid: "Your UUID"
    network: tcp
    tls: true
    udp: true
    flow: xtls-rprx-vision
    servername: "Your Domain"
    client-fingerprint: chrome
    reality-opts:
      public-key: "Your PublicKey"
      short-id: "Your ShortID"

proxy-groups:
  - name: PROXY
    type: select
    proxies:
      - Oracle-Reality
      - DIRECT

rules:
  # 本地及局域网地址直连。
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
  - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve

  # domain-list-community 广告域名。
  - GEOSITE,category-ads-all,REJECT

  # 指定域名强制代理的写法示例。
  - DOMAIN-SUFFIX,github.com,PROXY

  # 私有地址和中国大陆流量直连。
  - GEOSITE,private,DIRECT
  - GEOSITE,cn,DIRECT
  - GEOIP,CN,DIRECT,no-resolve

  # 未命中以上规则的流量统一代理；MATCH 必须放在最后。
  - MATCH,PROXY

```

