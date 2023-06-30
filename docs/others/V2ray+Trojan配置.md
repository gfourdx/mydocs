# V2ray+Trojan配置

参考: https://www.v2fly.org/config/overview.html

## 准备工作

- 服务端和客户端均下载[v2ray-core](https://github.com/v2fly/v2ray-core/releases/tag/v4.45.0)
- 客户端下载国内外路由分流文件[路由分流](https://github.com/Loyalsoldier/v2ray-rules-dat)

---

## 服务端

```json
{
  	// 配置日志
    "log": {
        "access": "access.log",
        "error": "error.log",
        "loglevel": "debug"
    },
    // 配置入站连接
    "inbounds": [
        {
          	// 使用Trojan协议
            "protocol": "trojan",
            "settings": {
                "clients": [
                    {
                        // 密码, 客户端认证使用
                        "password": "password"
                    }
                ]
            },
            "port": 56789,
            "listen": "0.0.0.0",
            "tag": "proxy" 
        }
    ],
    // 配置出站连接
    "outbounds": [
        {
            "protocol": "freedom"
        }
    ]
}
```

## 客户端

```json
{
     // 配置日志
    "log": {
        "access": "access.log",
        "error": "error.log",
        "loglevel": "debug"
    },
    // 配置入站连接
    "inbounds": [
        // 用于设置系统HTTP代理
        {
            "protocol": "http",
            "port": 8080,
            "listen": "0.0.0.0"
        },
      	// 用于设置系统SOCKS代理
        {
            "protocol": "socks",
            "settings": {
              	// 启用UDP协议
                "udp": true
            },
            "port": 1080,
            "listen": "0.0.0.0"
        }
    ],
  	// 配置出站连接
    "outbounds": [
        {
          	// 使用Trojan协议
            "protocol": "trojan",
            "settings": {
                "servers": [
                    {
                      	// 改成真正的服务器IP地址
                      	"address": "127.0.0.1",
                        // 服务器端口
                        "port": 56789,
                        // 服务端的密码, 
                        "password": "password"
                    }
                ]
            },
          	// 配置标签, 不能重复
            "tag": "proxy" 
        },
        {
            "protocol": "freedom",
            "tag": "direct" 
        }
    ],
  	// 配置路由, 实现分流
    "routing": {
      	// 只使用域名进行路由选择，默认值
        "domainStrategy": "AsIs",
      	// 域名匹配算法
        "domainMatcher": "mph",
      	// 路由规则
        "rules": [
            // 优先级从上往下依次递减
            {
                "domainMatcher": "mph",
                "type": "field",
                // 配置国内路由
                "domains": [
                    "geosite:private",
                    "geosite:cn",
                    "geosite:apple-cn",
                    "geosite:google-cn",
                    "geosite:category-games@cn",
                    // 避免 geolocation-!cn 导致 github 走代理
                    "github.com"
                ],
              	// 访问以上网站走直连
                "outboundTag": "direct"
            },
            {
                "domainMatcher": "mph",
                "type": "field",
                // 配置国外路由
                "domains": [
                    "geosite:google",
                    "geosite:facebook",
                    "geosite:twitter",
                    "geosite:telegram",
                    // geolocation-!cn 包含了常见的非大陆站点域名
                    // 启用后基本上所有国外网站(例如github)都走代理
                    "geosite:geolocation-!cn"
                ],
              	// 访问以上网站走代理
                "outboundTag": "proxy"
            },
            {
                // 其他网站走直连
                "type": "field",
                "network": "tcp,udp",
                "outboundTag": "direct"
            }
        ]
    }
}
```

## 其他

流量代理过程:

**浏览器 -> 系统代理(SOCKS) -> 客户端inbounds -> 客户端outbounds -> 服务端inbounds -> 服务端outbounds**

所以**客户端outbounds**配置的协议要于**服务端inbounds**一致, 否则无法代理成功

国内外分流路:

V2Ray 路由规则文件加强版，访问[路由分流](https://github.com/Loyalsoldier/v2ray-rules-dat)下载 `geoip.dat` 和 `geosite.dat` 的文件, 覆盖官方同名文件

## MAC脚本

适用于MacOS的一键脚本

```shell
#!/bin/zsh


getSOCKSProxyStatus(){
    status=`networksetup -getsocksfirewallproxy WI-FI|grep -E ^Enabled|cut -d " " -f 2`
}


enableSOCKSProxy(){
    `networksetup -setsocksfirewallproxy WI-FI 127.0.0.1 1080`
    `networksetup -setsocksfirewallproxystate WI-FI on`
    echo "Enable socks proxy for WI-FI"
}


disableSOCKSProxy(){
    `networksetup -setsocksfirewallproxystate WI-FI off`
    echo "Disable socks proxy for WI-FI"
}


startV2ray(){
    nohup ~/.v2ray/v2ray --config ~/.v2ray/config.json > ~/.v2ray/nohup.log 2>&1 &
    echo "V2ray started"
}


stopV2ray(){
    ps -ef|grep v2ray|grep -v v2ray.sh|grep -v grep|awk '{print $2}'|xargs kill -9
    echo "V2ray stopped"
}


main(){
    getSOCKSProxyStatus

    if [ $status == "Yes" ];
    then
        disableSOCKSProxy
        stopV2ray
    else
        startV2ray
        enableSOCKSProxy
    fi
}


main
```

PS: 配合alias别名`alias v2="sh ~/.v2ray/v2ray.sh"`使用效果更佳