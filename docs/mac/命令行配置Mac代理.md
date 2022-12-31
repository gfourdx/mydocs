# 命令行配置Mac代理

---

## 查看代理

### HTTP代理

语法:

`networksetup -getwebproxy <networkservice>`

示例:

```
networksetup -getwebproxy WI-FI
```

### HTTPS代理

语法:

 `networksetup -getsecurewebproxy <networkservice>`

示例:

```
networksetup -getsecurewebproxy WI-FI
```

### SOCKS代理

语法: `networksetup -getsocksfirewallproxy <networkservice>`

示例:

```
networksetup -getsocksfirewallproxy WI-FI
```

### 查看所有代理

```
# 方法1
scutil --proxy
# 方法2
system_profiler SPNetworkDataType|grep Proxy
```

## 设置代理

### HTTP代理

语法: `networksetup -setwebproxy <networkservice> <domain> <port number> <authenticated> <username> <password>`

示例:

```
# 配置 代理
networksetup -setwebproxy WI-FI 127.0.0.1 8080
# 配置 代理 + 认证
networksetup -setwebproxy WI-FI 127.0.0.1 8080 on username password
```

### HTTPS代理

语法: `networksetup -setsecurewebproxystate <networkservice> <domain> <port number> <authenticated> <username> <password>`

示例: 

```
# 配置 代理
networksetup -setsecurewebproxy WI-FI 127.0.0.1 8080
# 配置 代理 + 认证
networksetup -setsecurewebproxy WI-FI 127.0.0.1 8080 on username password
```

### SOCKS代理

语法: `networksetup -getsocksfirewallproxy <networkservice> <domain> <port number> <authenticated> <username> <password>`

示例:

```
# 配置 代理
networksetup -setsocksfirewallproxy WI-FI 127.0.0.1 1080
# 配置 代理 + 认证 
networksetup -setsocksfirewallproxy WI-FI 127.0.0.1 1080 on username password
```

## 启用|关闭 代理

### HTTP代理

语法: `networksetup -setwebproxystate <networkservice> <on off>`

示例:

```
# 启用代理
networksetup -setwebproxystate WI-FI on
# 关闭代理
networksetup -setwebproxystate WI-FI off
```

### HTTPS代理

语法: `networksetup -setsecurewebproxystate <networkservice> <on off>`

示例

```
# 启用代理
networksetup -setsecurewebproxystate WI-FI on
# 关闭代理
networksetup -setsecurewebproxystate WI-FI off
```

### SOCKS代理

语法: `networksetup -setsocksfirewallproxystate <networkservice> <on off>`

示例: 

```
# 启用代理
networksetup -setsocksfirewallproxystate WI-FI on
# 关闭代理
networksetup -setsocksfirewallproxystate WI-FI off
```