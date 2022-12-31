# Windows10离线安装.Net3.5

---

双击Windows10的ISO镜像进行挂载

记录挂载镜像的盘符

打开CMD输入以下命令

```
dism.exe /online /enable-feature /featurename:netfx3 /Source:F:\sources\sxs
```

> 注意：上方代码中`F:\sources\sxs`中的F即是挂载镜像的盘符