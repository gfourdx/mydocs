# UTM安装Windows

---

*为什么用UTM不用Parallels Desktop?*

看不惯PD收费模式, 而且总是碰到Windows时间变成2018年导致Mac时间也变成2018年的问题

都是基于QEMU, 但是UTM免费, 且提供两种模式**虚拟化**和**模拟**, 其中模拟方式不受CPU架构的限制

## 一、下载软件和镜像

- [UTM](https://github.com/utmapp/UTM/releases/download/v3.2.4/UTM.dmg) 是虚拟机软件

- [spice-guest-tools-0.164.3.iso](https://github.com/utmapp/qemu/releases/download/v6.2.0-utm/spice-guest-tools-0.164.3.iso) 提供主机和虚拟机之间复制、粘贴和共享等, 以及虚拟机屏幕自适应分辨率等功能

- [virtio-win-0.1.221.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.221-1/virtio-win-0.1.221.iso) 提供Windows对VirtIO磁盘的支持

- [UUP dump](https://uupdump.net/) 网站提供Windows镜像下载

  注意, UTM对系统版本有要求, 要求build number(**内部版本**)等于或高于21390, 否则安装过程中报错

  [**点击此处**](https://docs.microsoft.com/zh-cn/windows/release-health/windows11-release-information) 可获取最新的Window版本信息, 直接获取Windows11的最新正式版的内部版本后去 [UUP dump](https://uupdump.net/) 中搜索并下载

  说明一下, 正式版的Windows 10的内部版本均低于21390

  想要安装Windows 10只能使用Insider Preview中的Beta通道或Dev通道的版本(烦人的更新)

## 二、安装Windows11

### 步骤1

打开UTM -> 新建虚拟机  -> 虚拟化 -> Windows -> 取消勾选"导入 VHDX 磁盘镜像" -> 点击下方按钮"浏览" -> 选择Windows11 ISO镜像文件 -> 下一步 -> 下一步 -> 下一步 -> 下一步 -> 勾选"打开 VM 设置" -> 完成

> 上述4个下一步之间的3个过程中, 可配置配置内存和CPU、硬盘大小和共享文件夹功能, 默认或自定义均可

### 步骤2

找到**驱动器**分类下的**NVMe Deive** -> 单击 -> 接口 -> 选择"VirtIO" -> 保存

> 截止到2022年8月23日, UTM稳定版3.2.4不支持***暂停***对一个或多个驱动器使用NVMe接口的虚拟机
>
> 具体情况参见 [Windows 11 Mac M1 Pro - Pause Error · Issue #3918 · utmapp/UTM (github.com)](https://github.com/utmapp/UTM/issues/3918))
>
> 也就是说如果不改成VirtIO, 那么每次用完虚拟机后关闭虚拟机系统, 比较麻烦, 而VirtIO则支持***暂停***功能

### 步骤3

1、根据引导一步一步的安装Windows, 然后遇到问题"这台电脑无法运行 Windows 11"

2、此时, 按下**Shift+F10**, 在弹出的CMD窗口中输入regedit并回车, 定位到**HKEY_LOCAL_MACHINE\SYSTEM\Setup**

3、在**Setup**上右键 > 新建 > 项 > 命名为 **LabConfig**

4、在**LabConfig**上右键 > 新建 > DWORD (32 位)值 > 命名为 **BypassTPMCheck** > 打击打开该键值并输入 1 > 点击确定

5、在**LabConfig**上右键 > 新建 > DWORD (32 位)值 > 命名为 **BypassSecureBootCheck** > 打击打开该键值并输入 1 > 点击确定

6、关闭注册表 >关闭CMD >回到Windows安装页面 >点击左上角的返回箭头的按钮 > 点击我没有产品密钥 > 勾选接受条款 > 下一步

7、选择第二项(以"自定义:"开头), 发现没有可供Windows安装的硬盘

8、点击UTM软件右上角倒数第二个图标, 更换挂载的镜像为先前下载的**virtio-win-0.1.221.iso**, 然后回到Windows安装界面

9、点击左下角的"**加载驱动程序**" > **浏览** > CD驱动器  > 找到最下面的"**viostor**" > 找到最下面的"**w11**" > 点击"**ARM64**" > 点击**确定** > 点	击"**下一页**" > 等待驱动加载完毕会, 自动回到安装界面

10、点击UTM软件右上角倒数第二个图标, 更换挂载的镜像为先前下载的**Window 11 ISO安装镜像** > 点击左下角的"**刷新**" > 点击"**下一页**"  > 至此, 已经开始装安Windows 11

11、安装成功后, 经过设置进入Windows 11桌面后, 点击UTM软件右上角倒数第二个图标, 更换挂载的镜像为先前下载的**spice-guest-tools-0.164.3.iso**, 安装

12、搞定!