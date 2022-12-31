# 源码编译MySQL5.7记录

---

***Mac M1 Pro 芯片 安装Docker后想拉取MySQL5.7镜像, 奈何官方镜像中5.7版本只有amd64架构, 于是拉取了CentOS7.9.2009镜像, 在此环境下编译MySQL5.7, 做此记录***



纯粹的源码:

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-5.7.38.tar.gz

Red Hat使用的源码:

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-5.7.38-1.el7.src.rpm



纯源码安装--失败

```
cd
groupadd mysql
useradd -r -g mysql -s /bin/false mysql

yum install -y wget make cmake gcc gcc-c++ git \
  openssl-devel ncurses-devel bison curl-devel \


wget https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-5.7.38.tar.gz

tar -xf mysql-boost-5.7.38.tar.gz
cd mysql-5.7.38
mkdir bld
cd bld

cmake .. -DWITH_BOOST=../boost/boost_1_59_0 \
  -CMAKE_INSTALL_PREFIX=/opt/mysql \
  -DMYSQL_DATADIR=/opt/mysql/data \
  -DSYSCONFDIR=/opt/mysql \
  -DDEFAULT_CHARSET=utf8mb4 \
  -DDEFAULT_COLLATION=utf8mb4_general_ci \
  -DWITH_CURL=system \
  -DENABLE_DOWNLOADS=1 
  
注:
-DWITH_CURL=system 解决 You need to set WITH_CURL 问题
-DENABLE_DOWNLOADS=1 解决 Googletest was not found 问题

```



rpmbuild安装--成功

```
docker rm -f mysql

docker run -dit --name mysql --network host centos:7.9.2009 bash

docker cp mysql-community-5.7.38-1.el7.src.rpm mysql:/root

docker exec -it mysql bash

useradd pb2user

groupadd common

yum install -y rpm-build cmake perl-Env perl-Data-Dumper \
  perl-JSON time libaio-devel ncurses-devel numactl-devel \
  openssl-devel zlib-devel cyrus-sasl-devel openldap-devel \
  git make gcc gcc-c++

rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql
# rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

rpmbuild --rebuild --clean /root/mysql-community-5.7.38-1.el7.src.rpm


出现报错1:
/root/rpmbuild/BUILD/mysql-5.7.38/mysql-5.7.38/sql/mysqld.cc:1565:36: error: 'prctl' was not declared in this scope

则:
cp /usr/include/sys/prctl.h /root/rpmbuild/BUILD/mysql-5.7.38/mysql-5.7.38/include

vi /root/rpmbuild/BUILD/mysql-5.7.38/mysql-5.7.38/sql/mysqld.cc

在
#include "mysqld.h"
#include "mysqld_daemon.h"
下面添加:
#include "prctl.h"

cd /root/rpmbuild/BUILD/mysql-5.7.38/mysql-5.7.38

cmake . -DWITH_BOOST=../boost_1_59_0

make

出现报错2:
could not split insn
则:
yum -y remove gcc

yum install centos-release-scl

yum install devtoolset-7

scl enable devtoolset-7 bash

ln -s /opt/rh/devtoolset-7/root/usr/bin/c++ /usr/bin/c++

make

又出现报错1:
按照上面的方法解决

make

make install DESTDIR=/opt/mysql

编译安装成功

初始化
groupadd mysql

useradd -r -g mysql -s /bin/false mysql

cd /opt/mysql/usr/local/mysql

mkdir data

chown mysql:mysql data

chmod 750 data

mkdir etc

vi etc/my.cnf

写入:
[mysqld]
basedir=/opt/mysql/usr/local/mysql
datadir=/opt/mysql/usr/local/mysql/data


bin/mysqld --defaults-file=/opt/mysql/usr/local/mysql/etc/my.cnf \
  --initialize --user=mysql
  
注意1: 上面命令执行后输出的内容有一行类似这样的:
[Note] A temporary password is generated for root@localhost: a!mD0etq9!Mf
其中 a!mD0etq9!Mf 就是root用户的随机码尺码
注意2: --initialize 可以替换成 --initialize-insecure, 这样不会生成密码

PS: 如果不想创建 etc/my.cnf 则可以使用下面的命令
    bin/mysqld --initialize --user=mysql \
      --basedir=/opt/mysql/usr/local/mysql \
      --datadir=/opt/mysql/usr/local/mysql/data

启动服务:
bin/mysqld_safe --user=mysql &
或
bin/mysqld --defaults-file=/opt/mysql/usr/local/mysql/etc/my.cnf --user=mysql &

登录
bin/mysql -u root -p
如果上面用的是 --initialize-insecure, 则登录命令是
mysql -u root --skip-password

修改密码:
ALTER USER 'root'@'localhost' IDENTIFIED BY 'mysql';

查看用户信息:
select host, user, authentication_string, plugin from mysql.user;
注意: 高版本中authentication_string换成了password

关闭服务:
bin/mysqladmin -u root -p shutdown

搞定!

转移到其他centos7上是使用?
1、打包拷贝
cd /opt
tar -czvf mysql5.7.38.tgz mysql
2、解压
将压缩包拷贝到其他机器上并解压到/opt目录下
3、安装依赖: yum install -y libaio-devel numactl-devel
4、配置用户和组: groupadd mysql && useradd -r -g mysql -s /bin/false mysql
5、启动: cd /opt/mysql/usr/local/mysql , bin/mysqld_safe --user=mysql &
```

