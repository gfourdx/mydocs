# 手动安装WSL

参考：[[安装 WSL | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows/wsl/install)]

注解：以下命令均在CMD中操作

---

## 启用WSL功能

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

## 启用虚拟机功能

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## 下载并安装Linux内核更新包

[X64 包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

[ARM64 包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi)

## 设置WSL2为默认版本

```
wsl --set-default-version 2
```

## 下载发行版

- [Debian GNU/Linux](https://aka.ms/wsl-debian-gnulinux)
- [Ubuntu 22.04 LTS](https://aka.ms/wslubuntu2204)

> 更多发行版下载参见：[https://docs.microsoft.com/zh-cn/windows/wsl/install-manual#downloading-distributions](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual#downloading-distributions)

## 解压发行版

下载的Linux发行版文件格式是.appx，可直接右键使用7zip解压或修改扩展名为.zip后解压

解压到指定位置后（例如 D:\WSL\Debian），进入目录后创建双击“发行版名称.exe”即可安装

## 备注

同一个发行版默认只能安装一个实例，若想安装多个请按照以下步骤操作

1、先正常安装

2、打开注册表\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss

3、在右侧找到DistributionName

4、双击修改成其他名称，例如Debian1

5、重新解压双击exe即可安装多个实例

