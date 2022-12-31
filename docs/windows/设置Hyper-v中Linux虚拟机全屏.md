# 设置Hyper-v中Linux虚拟机全屏

---

## 编辑配置

```
vim /etc/default/grub
```

## 修改配置

找到有`GRUB_CMDLINE_LINUX_DEFAULT`的这行

修改为 `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video=hyperv_fb:1980x1080"`

其中 1980x1080 即是全屏的分辨率，请根据实际情况修改

## 刷新配置

### Debian系

```
sudo update-grub
sudo grub-mkconfig
```

### CentOS系

```
grub2-mkconfig -o /boot/grub2/grub.cfg 
```

然后重启系统即可

## 其他

如果上述方法没有生效，可能是需要安装一下依赖

```
# 1、先安装下linux核心和头文件
sudo apt-get install -y linux-image-3.11.0-15-generic linux-headers-3.11.0-15-generic
2、再安装核心文件扩展包
sudo apt-get install linux-image-extra-virtual
```

