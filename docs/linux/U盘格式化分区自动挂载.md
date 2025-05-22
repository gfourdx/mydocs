# 格式化重建分区

---

## 1、查找U盘

```
fdisk -l
```

假设找到U盘 /dev/sda

## 2、格式化ext4文件系统

```
mkfs.ext4 /dev/sda
```

输入`y`回车, 等待命令执行完毕

## 3、创建分区

```
fdisk /dev/sda
```

输入`n`新建分区

输入`p`创建主分区

输入`1`设置主分区序号

设置分区起始地址, 保持默认, 直接回车

设置分区结束地址, 保持默认, 直接回车

输入`w`保存并写入分区表

此时, 通过命令`fdisk -l`可看到分区`/dev/sda1`创建完毕

## 4、格式化分区

```
mkfs.ext4 /dev/sda1
```

## 5、更改U盘Label

```
e2label /dev/sda1 "mydiskname"
```

# 自动挂载U盘

## 1、查看U盘UUID

```
blkid|grep -i sda
```

得到类似这样的内容: `UUID="6f17b453-511e-4cb0-8655-ed487c51c54b"`

## 2、自动挂载

```
echo "UUID=6f17b453-511e-4cb0-8655-ed487c51c54b /mnt/sda1 ext4 defaults 0 0" >> /etc/fstab
```

解释:

/etc/fstab文件主要包括6段，依次是：

device　　mount point　　filesystem　　options　　dump　　pass

- device: 设备文件的UUID

- mount point: 挂载点, 即文件夹, 需要提前创建

- filesystem: 文件系统的格式，包括ext2、ext3、zfs、nfs、vfat等

- options: 如下

  | Defaults    | 同时具有rw,suid,dev,exec,auto,nouser,async等默认参数的设置   |
  | ----------- | ------------------------------------------------------------ |
  | Async/sync  | 设置是否为同步方式运行，默认为async                          |
  | auto/noauto | 当下载mount -a 的命令时，此文件系统是否被主动挂载。默认为auto |
  | rw/ro       | 是否以以只读或者读写模式挂载                                 |
  | exec/noexec | 限制此文件系统内是否能够进行"执行"的操作                     |
  | user/nouser | 是否允许用户使用mount命令挂载                                |
  | suid/nosuid | 是否允许SUID的存在                                           |
  | Usrquota    | 启动文件系统支持磁盘配额模式                                 |
  | Grpquota    | 启动文件系统对群组磁盘配额模式的支持                         |

- dump: dump工具备份文件系统的设置, 0表示忽略, 1表示备份

- Pass: fsck工具检查文件系统的设置, 0表示忽略,1表示获得最高优先权, 其他所有需要被检查的设备设置为2

## 3、扩展

手动挂载: `mount /dev/sda1 /mnt/usb`

手动卸载: `umount /dev/sda1`

