针对Debian12及以上版本的快速使用清华源的办法

```bash
sed -i 's@deb.debian.org@mirrors.tuna.tsinghua.edu.cn@g' /etc/apt/sources.list.d/debian.sources
```