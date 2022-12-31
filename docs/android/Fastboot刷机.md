# Fastboot刷机

1、正常启动手机, 开启调试模式

2、进入fastboot模式

```
adb reboot bootloader
```

3、检查设备

```
fastboot devices
```

4、根据实际情况擦除

```
必选擦除
fastboot erase system
fastboot erase cache
fastboot erase config
fastboot erase userdata

可选擦除
fastboot erase boot
fastboot erase recovery
```

5、刷机

```
fastboot flash boot boot.img
fastboot flash recovery recovery.img
fastboot flash system system.img
```

> boot.img和system.img需要提前从刷机包中解压出来, recovery.img一般是twrp.img

6、重启

```
fastboot reboot 
```

