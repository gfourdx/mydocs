# PIP

---

## 配置PIP清华源

### 命令配置

参考：[pypi | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)

```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 手动配置

#### 创建配置文件夹

```shell
mkdir -p ~/.config/pip
```

#### 创建配置文件

```shell
vim ~/.config/pip/pip.conf
```

#### 写入配置内容

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

## 下载包及其依赖到指定目录

```shell
pip download -d /path/to/target/ package
```

## 安装包及其依赖到指定目录

```shell
pip install --target=/path/to/target/ package
pip install --target=/path/to/target/ -r requirements.txt 
```

> 注意, 安装包到指定位置后会导致Python解释器无法加载这些包, 需要指定环境变量
>
> export PYTHONPATH=/path/to/target1;/path/to/target2;
