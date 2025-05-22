# WSL端口转发

---

## 增加端口转发

默认情况下wsl中的端口只能够被主机访问，若想要局域网内其他主机访问，则需要设置端口转发

命令格式：

netsh interface portproxy add v4tov4 listenaddress=监听地址 listenport=监听端口 connectaddress=被转发地址 connectport=被转发端口

命令详解：

- listenaddress 即局域网内其他主机访问的地址
- listenport 即局域网内其他主机访问的端口
- connectaddress 即WSL中的地址
- connectport 即WSL中的端口

示例：

```
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=5001 connectaddress=0.0.0.0 connectport=5000
```

## 删除端口转发

命令格式：

netsh interface portproxy delete v4tov4 listenaddress=监听地址 listenport=监听端口

示例：

```
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=5000
```

## 查看转口转发

若忘记转发的的地址和端口可使用以下命令查看

```
netsh interface portproxy show all
```
