# Iptables

---

## 一、Iptables结构

自上而下有由3部分组成:

- **Tables** - 表
- **Chains** - 链
- **Rules** - 规则

## 1、Tables组成

### 1.1、Tables

内建4中表, 分别是:

- **Filter** - 定义允许或不允许
- **NAT** - 定义地址转换
- **Mangle** - 修改报文原数据, 例如打标记
- **Raw** - ?

**优先级从上到下依次降低, 即: raw > mangle > nat > filter**

#### 1.2、Filter

Filter是Iptables的默认表,作用于以下3种内建链(Chains):

- **INPUT** - 数据包入口, 判断路由并处理来自外部的数据,
- **OUTPUT** - 数据包出口, 后接路由判断出口, 向外部发送的数据
- **FORWARD** - 不进入用户空间, 将数据转发到本机的其他网卡设备上

#### 1.3、NAT

NAT作用于3种内建链, 分别是:

- **PREROUTING** - 路由前, 数据包刚进入网络层
- **OUTPUT**
- **POSTOUTING** - 路由后, 数据包通过网络接口出去

#### 1.4、Mangle

Mangle则可以作用于以上全部的内建链:

- **PREROUTING**
- **INPUT**
- **FORWARD**
- **OUTPUT**
- **POSTOUTING**

#### 1.4、Raw

Raw只作用于2个内建链:

- **PREROUTING**
- **OUTPUT**

### 2、Chains

- **PREROUTING**
- **INPUT**
- **FORWARD**
- **OUTPUT**
- **POSTOUTING**

### 3、Rules

通过Iptables语法创建的规则

### 4、总结

通过Iptables语法创建的规则存放于Tables中并应用到Chains上面

---

## 二、Iptables语法

### 1、语法规则

格式：`iptables [-t table] COMMAND chain CRETIRIA -j ACTION`

`-t table`: filter、nat、mangle、raw

`COMMAND`:  定义如何对规则进行管理

 `chain`:  定义如何对规则进行管理, 指定该规则应用到哪个链上, 当定义策略(--policy)时可省略

`CRETIRIA`: 指定匹配标准

`-j ACTION`: 指定如何进行处理

### 2、参数说明

#### 2.1、COMMAND

##### 2.1.1、链管理命令

`-P`: 设置默认策略, 默认策略一般只有两种, 开(ACCEPT)或者关(DROP)

`-F`: 清空规则链, 但需要注意每个链的管理权限

`-N`: 新建一个自定义链

`-X`: 删除一个自定义链

`-E`: 重命名一个自定义链

`-Z`: 清空链及链中默认规则的计数器

##### 2.1.2、规则管理命令

`-A`: 在末尾增加一个规则

`-I num`: 把当前规则插入到第几条, num是数字, 省略num表示插入到最前面

`-R num`: 替换一个规则, 也可以理解为修改一个规则

`-D num`: 删除第几条规则

##### 2.1.3、查看命令

`-L`: 查看

`-n`: 显示为IP否则是域名

`-v`: 显示详细信息, 可以接多个v, v越多越详细, 例如 -vvv

`-x`: 在计数器上显示精确值, 不做单位转换

`-t table_name`: 显示指定表的信息, 例如 -t nat

##### 2.1.4、通用匹配命令

`-s`: 指定作为源地址匹配, 需使用IP, 不能是主机名, 格式包括IP、IP/MASK、0.0.0.0/0.0.0.0, 同时可以通过`!`取反

`-d`: 表示匹配目标地址

`-p`: 用于匹配协议, 常用的有TCP、UCP和ICMP等

`-i eth0`: 从eth0网卡流入的数据, 一般用于INPUT和PREOUTING上, eth0可以是其他接口

`-o eth0`: 从eth0网卡流出的数据, 一般用于OUTPUT和POSTOUTING上, eth0可以是其他接口

##### 2.1.4、扩展(隐式)匹配命令

当使用了通用匹配命令`-p`后, 针对不同的协议, 有不同的扩展匹配命令, 分别是:

TCP:

- --dport xx, xx是数字, 表示指定一个目标端口, 例如 --dport 22
- --dport xx-xx, 指定连续的目标端口, 例如 --dport 22-25
- --sport xx, 指定源端口
- --tcp-fiags：TCP的标志位(SYN, ACK, FIN, PSH, RST, URG), 对于它一般要跟两个参数: 检查的标志位和必须为1的标志位

UDP:

- --dport, 同TCP
- --sport, 同TCP

ICMP:

- --icmp-type: echo-request(请求回显), 一般用8 来表示, 所以 --icmp-type 8 匹配请求回显数据包; echo-reply （响应的数据包）一般用0来表示

##### 2..1.5、显示匹配命令

用于开展各种模块, 常用的是`-m`, 即mulitport, 表示启用多端口扩展, 比如 --dports 21,23,80

#### 2.2、chain

#### 2.3、CRETIRIA

#### 2.4、-j ACTION

常用的ACTION分别是:

- DROP - 悄悄丢弃
- REJECT - 明确拒绝
- ACCEPT - 接受
- REDIRECT - 重定向, 主要用于实现端口重定向
- MARK - 打防火墙标记

其他的还有:

- custom_chain - 转向一个自定义的链
- RETURN - 返回, 在自定义链执行完毕后使用返回来返回原规则链
- DNAT
- SNAT
- MASQUERADE - 是 SNAT 的一种特殊形式，适用于动态的、临时会变的 ip 上, 例如源地址伪装
- LOG

### 3、永久保存

通过iptables命令添加、删除的规则不会永久生效, 重启后恢复原样, 持久化的命令为:

Debian系列: service iptables save 或 /sbin/iptables-save

RedHat系列: /sbin/service iptables save

查看: cat  /etc/sysconfig/iptables