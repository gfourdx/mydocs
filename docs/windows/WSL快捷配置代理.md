# WSL快捷配置代理

编辑.profile文件并写入以下内容：

```bash
# 定义 clash 函数
clash() {
    # 检查是否设置了代理
    if [ -z "$https_proxy" ] && [ -z "$http_proxy" ] && [ -z "$all_proxy" ]; then
        # 如果没有开启代理，则设置代理
        WINHOST=$(ip route show | grep -i default | awk '{ print $3 }')
        export https_proxy=http://$WINHOST:7890 http_proxy=http://$WINHOST:7890 all_proxy=socks5://$WINHOST:7890
        echo "Proxy enabled"
    else
        # 如果已经开启代理，则取消代理
        unset https_proxy http_proxy all_proxy
        echo "Proxy disabled"
    fi
}

# 设置别名 clash
alias clash=clash
```

