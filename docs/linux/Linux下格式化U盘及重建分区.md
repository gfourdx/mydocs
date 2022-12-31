# Linux下格式化U盘及重建分区

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

