# 基于Docker的开发环境 V2

---

```dockerfile
#
# 构建Docker开发环境
# 
# 1、构建镜像
# docker build -f dev.Dockerfile -t gfourdx/dev:latest . 
# 2、运行容器
# docker run -dit -h dev -p 2222:22 -p 80:80  -p 443:443 -p 3306:3306 -p 5432:5432 -p 6379:6379 --name dev --restart=unless-stopped gfourdx/dev:latest
# 3、快速进入容器
# echo 'alias dev="docker exec -it dev bash"' >> ~/.zprofile
# 
# 利用多段构建构建解决两个问题:
# 1、因缺少编译工具导致 pip 命令安装某些包时失败的问题
# 2、减少镜像的体积
#

# 第一阶段构建
FROM debian:bookworm-slim AS base

# 配置编译环境
# 每条命令之间使用 && 连接
# 表示只有上条命令执行成功后才会执行下条命令
RUN set -eux; \
    # 配置国内源
    echo "Types: deb" > /etc/apt/sources.list.d/debian.sources && \
    echo "URIs: http://mirrors.tuna.tsinghua.edu.cn/debian" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Suites: bookworm bookworm-updates bookworm-backports" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Components: main contrib non-free non-free-firmware" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg" >> /etc/apt/sources.list.d/debian.sources && \
    echo "" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Types: deb" >> /etc/apt/sources.list.d/debian.sources && \
    echo "URIs: http://mirrors.tuna.tsinghua.edu.cn/debian-security" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Suites: bookworm-security" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Components: main contrib non-free non-free-firmware" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg" >> /etc/apt/sources.list.d/debian.sources && \
    # 更新源
    apt-get update

RUN set -eux; \
    # 安装包
    apt-get install --no-install-recommends --no-install-suggests -y \
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
        default-libmysqlclient-dev \
        libpq-dev && \
    # 清除缓存
    rm -rf /var/lib/apt/lists/*

# 配置虚拟环境并pip安装编译包
RUN set -eux; \
    # 配置国内源
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    # 创建虚拟环境
    python3 -m venv /root/venv && \
    # 激活虚拟环境
    # dockerfile中source语法无效
    # 使用 . 代替
    . /root/venv/bin/activate && \
    # 安装wheel
    pip install wheel && \
    # 安装需要编译的包
    pip install mysqlclient psycopg2

# 第二阶段构建
FROM debian:bookworm-slim

# 配置环境变量
ENV LANG=C.UTF-8

# 配置工作目录
WORKDIR /root/

RUN set -eux; \
    # 配置本地时间
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    # 配置本地时区
    echo 'Asia/Shanghai' > /etc/timezone

RUN set -eux; \
    # 配置国内源
    echo "Types: deb" > /etc/apt/sources.list.d/debian.sources && \
    echo "URIs: http://mirrors.tuna.tsinghua.edu.cn/debian" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Suites: bookworm bookworm-updates bookworm-backports" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Components: main contrib non-free non-free-firmware" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg" >> /etc/apt/sources.list.d/debian.sources && \
    echo "" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Types: deb" >> /etc/apt/sources.list.d/debian.sources && \
    echo "URIs: http://mirrors.tuna.tsinghua.edu.cn/debian-security" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Suites: bookworm-security" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Components: main contrib non-free non-free-firmware" >> /etc/apt/sources.list.d/debian.sources && \
    echo "Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg" >> /etc/apt/sources.list.d/debian.sources && \
    # 更新源
    apt-get update

RUN set -eux; \
    # 安装包
    apt-get install --no-install-recommends --no-install-suggests -y \
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
    rm -rf /var/lib/apt/lists/* && \
    # 创建虚拟环境
    python3 -m venv venvs/venv

RUN set -eux; \
    # 配置PIP国内源
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    # 配置mariaDB监听所有IP
    sed -i 's/127.0.0.1/0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf && \
    # 配置PostgreSQL允许任何IP登录
    sed -i '0,/127.0.0.1\/32/s//0.0.0.0\/0/' /etc/postgresql/15/main/pg_hba.conf && \
    # 配置PostgreSQL监听所有IP
    sed -i "0,/#listen_addresses = 'localhost'/s//listen_addresses = '*'/" /etc/postgresql/15/main/postgresql.conf && \
    # 配置redis监听所有IP
    sed -i '0,/^bind 127.0.0.1/s//bind 0.0.0.0/' /etc/redis/redis.conf && \
    # 配置redis非保护模式(允许非回环地址访问)
    sed -i "s/protected-mode yes/protected-mode no/" /etc/redis/redis.conf&& \
    # 配置ssh允许root登录
    sed -i "s/#PermitRootLogin prohibit-password/PermitRootLogin yes/" /etc/ssh/sshd_config && \
    # 配置ssh允许空密码登录
    sed -i "s/#PermitEmptyPasswords no/PermitEmptyPasswords yes/" /etc/ssh/sshd_config && \
    # 删除root密码
    passwd -d root

# 从第一阶段拷贝虚拟环境中的包
COPY --from=base /root/venv/lib/python3.11/site-packages/ venvs/venv/lib/python3.11/site-packages/

# 22 ssh
# 80 443 nginx
# 3306 mariadb
# 5432 postgresql
# 7379 redis
EXPOSE 22 80 443 3306 5432 6379

# 通过 bash -c 参数的方式启动服务
# 最后使用 bash 或 tail -f /dev/null 挂起
# 保证容器运行后不会直接退出
# CMD ["bash", "-c", "service server1 start && service server2 start && xxx && tail -f /dev/null"]
CMD ["bash", "-c", "service ssh start && service nginx start && service mariadb start && service postgresql start && service redis-server start && bash"]
```
