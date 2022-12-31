# KMS激活

---

## 查询KMS密钥

### Windows密钥地址

[https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys](https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys)

### Office密钥地址

[https://docs.microsoft.com/zh-cn/DeployOffice/vlactivation/gvlks](https://docs.microsoft.com/zh-cn/DeployOffice/vlactivation/gvlks)

### 常用密钥

| 版本                                                        | 密钥                          |
| :---------------------------------------------------------- | :---------------------------- |
| Windows 11 专业版<br/>Windows 10 专业版                     | W269N-WFGWX-YVC9B-4J6C9-T83GX |
| Windows 11 企业版<br/>Windows 10 企业版                     | NPPR9-FWDCX-D2C8J-H872K-2YT43 |
| Windows 11 教育版<br/>Windows 10 教育版                     | NW6C2-QMPVW-D7KKK-3GKT6-VCFB2 |
| Windows 10 企业版 LTSC 2021<br/>Windows 10 企业版 LTSC 2019 | M7XTQ-FN8P6-TTKYV-9D4CC-J462D |
| Office LTSC 专业增强版 2021                                 | FXYTK-NJJ8C-GB6DW-3DYQT-6F7TH |
| Office LTSC 标准版 2021                                     | KDX7X-BNVR8-TXXGX-4Q7Y8-78VT3 |
| Office 专业增强版 2019                                      | NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP |
| Office 标准版 2019                                          | 6NWWJ-YQWMR-QKGCB-6TMB3-9D9HK |



## 使用KMS激活

以管理员身份运行CMD

### Windows激活

在CMD中依次输入以下命令

```
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX    # 安装密钥
slmgr /skms kms.03k.org                     # 设置KMS主机
slmgr /ato                                  # 激活
slmgr /dlv                                  # 查看激活状态
```

### Office激活

在CMD中依次输入以下命令

```
cd C:\Program Files\Microsoft Office\Office16             # 进入Office脚本目录
cscript ospp.vbs /inpkey:FXYTK-NJJ8C-GB6DW-3DYQT-6F7TH    # 安装密钥
cscript ospp.vbs /sethst:kms.03k.org                      # 设置KMS主机
cscript ospp.vbs /act                                     # 激活
cscript ospp.vbs /dstatus                                 # 查看激活状态
```

> 关于office，如果上面方法无法激活可尝试如下方法，将以下代码保存为.bat文件，然后双击执行：
>
> @echo off
>
> cd C:\Program Files\Microsoft Office\Office16<br/>
> cscript ospp.vbs /sethst:kms.03k.org<br/>
> cscript ospp.vbs /act<br/>
> cscript ospp.vbs /dstatus<br/>
> cscript ospp.vbs /remhst<br/>
>
> pause



## 其他

kms.03k.org 是公共的KMS激活主机，类似的还有 kms.shax.vip 和 91.149.135.83

若上述命令无法激活可尝试更换KMS主机