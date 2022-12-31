# RHEL7.3的Jira安装与破解教程

---

## 一、下载并安装Jira
访问 Jira 官方下载地址：https://www.atlassian.com/software/jira/download

下载 Enterprise release (企业版)，目前版本为 Jira Software 8.5.4，选择下载 Linux 64 Bit

得到二进制安装文件：atlassian-jira-software-8.5.4-x64.bin

1.1、执行命令赋予安装包执行权限：
```
chmod a+x atlassian-jira-software-8.5.4-x64.bin
```
1.2、根据官方文档提示安装依赖：https://confluence.atlassian.com/jirakb/jira-server-7-13-or-later-fails-with-fontconfiguration-error-when-installing-on-linux-operating-systems-964956221.html
```
sudo yum install -y dejavu-sans-fonts  (已安装的请跳过，未安装的请安装，否则下一步命令报错)
```
1.3、开始安装：
```
./atlassian-jira-software-8.5.4-x64.bin
```
1.4、一路回车默认即可，但是最后一步选择no（先不启动 Jira）

## 二、查阅Jira安装要求
2.1、查阅 Jira 官方文档：https://confluence.atlassian.com/adminjiraserver085/supported-platforms-981154553.html

得知支持的 PostgreSQL 版本为 9.4-10

2.2、RHEL7.3仓库中 PostgreSQL 的版本为9.2，无法被 Jira 支持

## 三、安装及配置数据库
3.1、查阅 PostgreSQL 官方文档：https://www.postgresql.org/download/linux/redhat/

根据页面指引版本选择"10"，平台选择"RedHat Enterprise, CentOS, Scientific or Oracle version 7"，架构选择"x86_64"

3.2、然后依次执行安装命令：

```
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install postgresql10
yum install postgresql10-server
```
3.3、安装成功后进行初始化设置及设置为开机自启
```
/usr/pgsql-10/bin/postgresql-10-setup initdb
systemctl enable postgresql-10
systemctl start postgresql-10
```

> PS1：如果之前已经安装了低版本的 PostgreSQL，请备份数据库后卸载

>列出已安装包名：rpm -qa | grep postgres 

> 删除已安装包：rpm -e 包名

> 检查残留文件并删除（可选）：rpm -qal | grep postgres

> ** PS2：记得去目录 /var/lib/pgsql/10/data 下修改连接参数以便可以远程登录pg **

> PS3：也可自行安装其他受支持的数据库

## 四、破解Jira
4.1、打开网址：https://gitee.com/pengzhile/atlassian-agent/releases

下载破解补丁 atlassian-agent-v1.2.3.tar.gz 并解压得到jar包 atlassian-agent.jar 和使用帮助文件 README.pdf

4.2、根据 README.pdf 文件指引，复制破解工具到指定目录：
```
cp atlassian-agent.jar /opt/atlassian
```
4.3、编辑文件 /opt/atlassian/jira/bin/setenv.sh，在文件最后添加以下内容：
```
export JAVA_OPTS="-javaagent:/opt/atlassian/atlassian-agent.jar ${JAVA_OPTS}" 
```

## 五、初始化配置Jira
5.1、启动 Jira：
```
/opt/atlassian/jira/bin/start-jira.sh
```
5.2、通过 IP:Port 方式打开 Jira配置页面，例如本示例中为 http://10.10.10.40:8080/

5.3、选择第二项 I'll set it up myself 并点击下一步

5.4、选择第二项 My Own Database，配置数据库信息并点击下一步

5.5、配置 Jira 属性，可编辑 Application Title 和 Mode,建议修改 Title后直接点击下一步

5.6、此时到了激活页面，记录下 Server ID，利用前面的破解工具生成License：
```
/opt/atlassian/jira/jre/bin/java -jar /opt/atlassian/atlassian-agent.jar -p jira -m wanglong@wntime.com -n WNTIMEJIRA -o http://www.wntime.com/ -s BNEV-U8I4-P1DW-AJNK
```
5.7、将生成的License粘贴进去并点击下一步

5.8、创建账户信息并点击下一步

5.9、邮件通知默认稍后配置，直接点击完成

5.10、选择中文，点击继续再点击下一步，至此 Jira安装破解完成

## 六、破解 Jira 插件
直接执行命令（同5.6），只需修改 -p 后面的参数，具体请参考破解工具的帮助文件 README.pdf，此处不再赘述！