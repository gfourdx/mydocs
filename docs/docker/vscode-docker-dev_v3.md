# 基于Docker的开发环境 V3

---

```dockerfile
FROM debian:trixie-slim

# 设置语言和编码
ENV LANG=C.UTF-8

# 配置本地时间和本地时区
RUN set -eux; \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' > /etc/timezone

# 设置工作目录
WORKDIR /root

# 换源、更新、安装基础软件、清理缓存、设置别名
RUN sed -i 's@deb.debian.org@mirrors.tuna.tsinghua.edu.cn@g' /etc/apt/sources.list.d/debian.sources \
    && apt-get update \
    && apt-get install -y --no-install-recommends --no-install-suggests \
       vim git openssh-server curl iproute2 iputils-ping procps ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && echo "alias agi='apt-get install --no-install-recommends --no-install-suggests'" >> /root/.bashrc \
    && echo "alias ll='ls -ahlrt'" >> /root/.bashrc

# 安装uv安装python配置pip源
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
RUN /bin/uv python install
ENV UV_DEFAULT_INDEX="https://pypi.tuna.tsinghua.edu.cn/simple"

# 关键：必须创建 /run/sshd 目录，否则服务无法启动
# 设置root空密码登录
RUN mkdir -p /run/sshd \
    && sed -i "s/#PermitRootLogin prohibit-password/PermitRootLogin yes/" /etc/ssh/sshd_config \
    && sed -i "s/#PermitEmptyPasswords no/PermitEmptyPasswords yes/" /etc/ssh/sshd_config \
    && passwd -d root

# 开放端口
EXPOSE 22

# 使用 -D 模式在前台运行 sshd，保证容器不退出
CMD ["/usr/sbin/sshd", "-D"]

# docker build -t deve .
# docker run -d --name debian --restart unless-stopped -h debian -p 22:22 deve
```
