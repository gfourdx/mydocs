# 基于Docker的Python开发环境

---

```dockerfile
FROM debian:bullseye-slim

ENV LANG C.UTF-8

RUN set -eux; \
    # 设置时区
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    # 配置apt源
    echo 'deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free \n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free \n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free \n\
deb http://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free' > /etc/apt/sources.list; \
    # 更新源
    apt update; \
    # 安装系统软件
    # iproute2提供`ip a`和`ss`命令, iputils-ping提供`ping`命令, procps提供`ps`和`top`命令
    apt install -y --no-install-recommends --no-install-suggests \
        iproute2 \
        iputils-ping \
        procps \
    ; \
    # 安装工具软件
    apt install -y --no-install-recommends --no-install-suggests \
        curl \
        git \
        ssh \
        wget \
        vim \
   ; \
   # 安装开发软件
    apt install -y --no-install-recommends --no-install-suggests \
        mariadb-client \
        mariadb-server \
        nginx \
        postgresql \
        python3 \
        python3-pip \
        python3-venv \
        redis \
   ; \
    # 避免pip安装mysqlclient和psycopg2报错
    apt install -y --no-install-recommends --no-install-suggests \
        build-essential \
        libmariadb-dev \
        libpq-dev \
        python3-dev \
    ; \
    # 清理缓存
    rm -rf /var/lib/apt/lists/*; \
    # 配置pip源
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple; \
    # 配置ssh允许root登录
    sed -i "s@#PermitRootLogin prohibit-password@PermitRootLogin yes@g" /etc/ssh/sshd_config; \
    # 配置ssh允许空密码登录
    sed -i "s@#PermitEmptyPasswords no@PermitEmptyPasswords yes@g" /etc/ssh/sshd_config; \
    # 删除root密码
    passwd -d root; \
    # 设置ssh自启动
    echo "service ssh start > /dev/null" >> /root/.bashrc
    
EXPOSE 22 80 3306 5432 6379

CMD ["bash" ]

# docker build -f Pydev.Dockerfile -t pydev:latest .
# docker run -dit -h pydev -p 22:22 -p 80:80 -p 3306:3306 -p 5432:5432 -p 6379:6379 -v /path/to/host:/path/to/docker --restart=unless-stopped --name pydev pydev
# docker exec -it pydev bash
```

