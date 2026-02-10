# VSCode免密发布代码到GitHub和配置443端口访问

## 设置Git全局变量

```
git config --global user.name "your name"     
git config --global user.email "your email"
```

## 生成SSH Key

```
ssh-keygen -t ed25519 -C "your email"
```

或

```
ssh-keygen -t rsa -b 4096 -C "your email"
```

### 查看公钥

```
cat ~/.ssh/id_ed25519.pub
```

或

```
cat ~/.ssh/id_rsa.pub
```

### 添加公钥到GitHub账号

setting -> SSH and GPG keys -> New SSH key

粘贴密钥并保存

## 发布代码

打开vscode，打开源代码管理，点击发布到Github，根据提示操作即可

## 配置443端口

某些情况下无法发现通过ssh克隆仓库失败, 但是ping github.com却是痛的
此时可能需要配置使用443端口访问, 即: `vim ~/.ssh/config`, 写入:
```
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
``` 
最后配置权限: `chmod 644 ~/.ssh/config`
