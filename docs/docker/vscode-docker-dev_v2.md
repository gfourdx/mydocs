# 基于Docker的开发环境 V2

---

```dockerfile
FROM debian:bookworm-slim as Base

WORKDIR /root/

RUN set -eux; \
    # 配置国内源
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware" > /etc/apt/sources.list; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware" >> /etc/apt/sources.list; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware" >> /etc/apt/sources.list; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware" >> /etc/apt/sources.list; \
    # 更新源
    apt update && \
    # 安装包
    apt install --no-install-recommends --no-install-suggests -y \
    # python相关的包
    python3 \
    python3-pip \
    python3-venv \
    python3-dev \
    \
    # 编译需要的包
    pkg-config \
    gcc \
    \
    # mysqlclient和psycopg2依赖的包
    libmariadb-dev \
    libpq-dev && \
    # 清除缓存
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    # 配置国内源
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    python3 -m venv venvs/venv && \
    # dockerfile中source语法无效
    # 使用 . 代替
    . venvs/venv/bin/activate && \
    pip install wheel && \
    pip install mysqlclient psycopg2;


FROM debian:bookworm-slim

ENV LANG C.UTF-8

WORKDIR /root/

RUN set -eux; \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' > /etc/timezone

RUN set -eux; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware" > /etc/apt/sources.list; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware" >> /etc/apt/sources.list; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware" >> /etc/apt/sources.list; \
    echo "deb http://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware" >> /etc/apt/sources.list; \
    apt update && \
    apt install --no-install-recommends --no-install-suggests -y \
    python3 \
    python3-pip \
    python3-venv \
    \
    mariadb-server \
    mariadb-client \
    postgresql \
    \
    redis \
    \
    nginx \
    \
    git \
    ssh \
    vim \
    \
    curl \
    wget \
    \
    # 提供ip xx和ss命令
    iproute2 \ 
    # 提供ping等命令
    iputils-ping \
    # 提供ps、top、free和sysctl等命令
    procps && \
    # 清理缓存
    rm -rf /var/lib/apt/lists/*;

RUN set -eux; \
    # 配置PIP国内源
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    # 配置mariaDB监听所有IP
    sed -i 's@bind-address            = 127.0.0.1@bind-address            = 0.0.0.0@g' /etc/mysql/mariadb.conf.d/50-server.cnf && \
    # 配置PostgreSQL允许任何IP登录
    sed -i "s@host    all             all             127.0.0.1/32            scram-sha-256@host    all             all             0.0.0.0/0               scram-sha-256@g" /etc/postgresql/15/main/pg_hba.conf && \
    # 配置PostgreSQL监听所有IP
    sed -i "s@#listen_addresses = 'localhost'		# what IP address(es) to listen on;@listen_addresses = '*'	# what IP address(es) to listen on;@g" /etc/postgresql/15/main/postgresql.conf && \
    # 配置ssh允许root登录
    sed -i "s@#PermitRootLogin prohibit-password@PermitRootLogin yes@g" /etc/ssh/sshd_config && \
    # 配置ssh允许空密码登录
    sed -i "s@#PermitEmptyPasswords no@PermitEmptyPasswords yes@g" /etc/ssh/sshd_config && \
    # 删除root密码
    passwd -d root; 

COPY --from=Base /root/venvs/ venvs/

EXPOSE 22 443  80 3306 5432 6379

# 通过 bash -c 参数的方式启动服务
# 最后使用 && bash 或 && tail -f /dev/null 挂起
# 保证容器运行后不会直接退出
# CMD ["bash", "-c", "service name1 start && service name2 start && xxx && tail -f /dev/null"] 
CMD ["bash", "-c", "service ssh start && service mariadb start && service postgresql start && service redis-server start && service nginx start && bash"]


# 构建镜像
# docker build -f mydev.Dockerfile -t mydev:latest . 
# 运行容器
# docker run -dit -h mydev -p 22:22 -p 80:80  -p 443:443 -p 3306:3306 -p 5432:5432 -p 6379:6379 -v /path/to/host:/path/to/docker --name mydev --restart=unless-stopped mydev
# 快速进入容器
# echo 'alias mydev="docker exec -it mydev bash"' >> ~/.zprofile
```
