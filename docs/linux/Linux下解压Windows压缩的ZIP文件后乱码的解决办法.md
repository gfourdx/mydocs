# Linux下解压Windows压缩的ZIP文件后乱码的解决办法

---

**问题原因**

Windows下压缩zip文件使用的是CP936编码

>CP936编码就是GBK，当IBM发明代码页时，把GBK放在了第936页，所以GBK又被称为CP936。

**解决办法**

`unzip -O CP936 xxx.zip`