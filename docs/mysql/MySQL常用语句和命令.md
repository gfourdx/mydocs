# MySQL常用语句和命令

---

## DDL相关基础语句

DDL 即数据库定义语言(data definition language)

### 显示数据库

```sql
show databases;
```

### 创建数据库

```sql
create database `db` character set 'utf8mb4' collate 'utf8mb4_general_ci';
```

### 删除数据库

```sql
drop database `db`;
```

### 切换数据库

```sql
use `db`;
```

### 查看当前选择的数据库

```sql
select database();
```

## DML相关基础语句

DML 即数据操纵语言(data manipulation language)

### 查看表

```sql
show tables;
```

### 创建表

语法:  `create table 表名(字段、类型、约束);` , 例如:

```sql
create table students(
		id int auto_increment primary key not null,
		name varchar(10) not null,
		sex bit default 1,
		birthday datetime);
```

> 扩展:
>
> 查看表的创建语句 - `show create table 表名`;
>
> 查看表结构：`desc 表名;`

### 修改字段

语法:  `alter table 表名 add|change|drop 列名 类型;` , 例如：

```sql
-- 新增字段is_delete, bit类型, 默认0
alter table students add is_delete bit default 0;

-- 修改字段
alter table students change is_delete delete_flag smallint default 0;

-- 删除字段
alter table students drop delete_flag;
```

### 修改表名

语法: `rename table 原表名 to 新表名;` , 例如:

```sql
rename table students to person;
```

### 删除表

语法: `drop talbe 表名1,表名2,表名3;` , 例如:

```sql
drop table person;
```

### 查询数据

语法: `select * from 表名 where 条件;`或`select 列1,列2,...列n from 表名 where 条件` , where条件可省略, 例如:

```sql
select * from students;
select name from students;
select * from students where birthday<'2000-01-01';
```

### 插入数据

#### 全部列插入

语法: `insert into 表名 values(列1, 列2, ... 列n);` , 例如:

```sql
/*
0 代指主键, 主键自增, 此处用0, 系统自动分配主键
如果输入的不是0, 则插入, 但之后主键以此插入的值为基数递增(不建议手动指定, 可能会出现跳号)
values 也可以写成 value, 同时括号内的数据要和字段一一对应
*/
insert into students values(0, '张三', 1, '2001-01-01');
```

#### 部分列插入

```sql
insert into students(name, birthday) values('李四', '2001-01-01 01:02:03.456789');
```

#### 多记录插入

```sql
-- 用逗号隔开多条记录即可
insert into students value(0, '王朝', 1, '2001-01-01'),(0, '马汉', 1, '2001-01-01');
```

### 修改数据

语法: `update 表名 set 列1=值1,列2=值2... where 条件;` , where条件可省略, 例如:

```sql
update students set name='王五' where name='张三';
update students set name='赵六', sex=0 where name='王五';
update students set birthday='2022-01-01' where birthday>'2022-12-31';
```

### 删除数据

语法: `delete from 表名 where 条件;` , where条件可省略, 例如:

```sql
delete from students where name='李四';
delete from students;
```

## DCL相关基础语句

DCL 即数据库控制语言(Data Control Language)

```

```

## 条件语句和其他常见语句

```sql
-- 范围
select * from students where age>=20;
delete from students where '2000-01-01'<birthday<='2000-12-31';
select * from students where sex in (0,1,2);
select * from students where id between 3 and 5;

-- 与或非
-- 注意 != 的另一种写法是 <>
select * from students where name='王朝' and age!=20 or sex not in (1,2,3);

-- 优先级
-- 通过括号改变and or 的优先级
select * from students where name='王朝' and (age!=20 or sex not in (1,2,3));

-- 空判断
select * from students where birthday is null;
select * from students where sex is not null;

-- 模糊查询
-- % 表示0个或多个任意字符, _ 表示一个任意字符
select * from students where name like '%三';
select * from students where name like '_三';
select * from students where name like '_三%'

-- 分组与聚合
select count(*) from students; -- 统计有多少行
select max(age) from students group by sex;  -- 查询男女中最大的年龄
select min(age) from students group by sex;  -- 查询男女中最小的年龄
select avg(id) from students;  -- 查询id的平均数
select sum(id) from students;  -- 查询id的总和

-- as 别名
select sex as gender from students;

-- having
-- having 用于where条件后, 用于进一步过滤筛选
select * from students where birthday>='2001-1-2' having name='b';
select * from students where birthday>='2001-1-2' and name='b';  -- 结果同上

-- 排序
select * from students order by id;  -- 升序
select * from students order by id asc;  -- 升序
select * from students order by id desc;  -- 降序
select * from students order by sex,id desc;  -- 先按sex降序, 在此基础上再按id降序

-- 分页
select * from students limit 1，3;  # 显示第2条到第3条数据(索引从0开始)
-- 每页显示 m 条数据，当前显示第 n 页。
select * from students limit(n-1)*m, m;

-- 去重
select distinct name from students;

```

## 外键约束

创建语法: `lter table 从表 add constraint 外键名称 foreign key (被约束字段) references 主表(约束字段);`

```sql
alter table scores add constraint fk_sut_sco foreign key(stuid) references students(id);
alter table scores add constraint fk_sut_sub foreign key(stuid) references students(id);
```

> 创建表时同时添加外键约束的语法示例:
>
> create table scores(id int auto_increment primary key not null,
> 			   stuid int,
> 			   subid int,
> 			   scores decimal(5,2),
> 			   foreign key(stuid) references students(id),
> 			   foreign key(subid) references subjects(id)
> 			  );

删除语法: `alter table 表名 drop foreign key 外键约束名;`

```
alter table scores drop foreign key fk_sut_sub;
```

扩展, 外键异常:

- restrict(限制)：默认值，即默认方式，抛出异常
- cascade(级联)：如果主表的记录删除，则从表中相关联的记录都将被删除
- set null：将外键设置为空
- no action：什么都不做

**restric方式抛出异常, 不友好; cascade方式物理删除数据, 产生连锁反应; 后两中方式产生冗余数据; 建议使用逻辑删除, 毕竟数据无价;**

## 关联查询

例如: 

学生信息存在表 students，科目信息存在表 subjects，分数信息存在 scores

scores 表存在两个外键 fk_sut_sco 和 fk_sut_sub , 分别关联表 students 和表 subjects

语法: `select * from 表 inner join 表 on 关系;`

```sql
select students.name,subjects.title,scores.scores
	from scores
	inner join students on scores.stuid=students.id
	inner join subjects on scores.subid=subjects.id;

-- 这种写法也是可以的，虽然表students和表subjects没有直接关系，但是通过scores表也可以关联
select students.name,subjects.title,scores.scores
		from students
		inner join scores on students.id=scores.stuid
		inner join subjects on subjects.id=scores.subid;
```

除了 inner join 还有 left join 和 right join, 区别是:

- 表A inner join 表B 会显示出两张表共有的数据
- 表A left join 表B 除了会显示两张表共有的数据外还会显示表A独有的数据, 未对应的数据使用null填充
- 表A righ join 表B 除了会显示两张表共有的数据外还会显示表B独有的数据，未对应的数据使用null填充

**省略了 inner join ... on ... 的关联查询的写法:**

```sql
select name,title,score from students,subjects,scores where scores.stuid=students.id and scores.subid=subjects.id;
```

**这种写法省略了 inner join ... on ..., 直接在from后用逗号隔开表明, where后面的条件写上各个表的关系, where 后面的关系一定要写上且上写对, 否则查出来的数据可能是错误的**

## 视图

举例, 每次查询学生的 ID、姓名、科目、成绩, 都需要做关联查询, 非常麻烦, 所以可以将这句代码封装起来, 做成视图
语法: `create view 视图名称 as 查询语句;`

```sql
-- 创建视图
create view v_cfs as select students.id,name,title,score from scores inner join students on stuid=students.id inner join subjects on subid=subjects.id;

-- 用法
select * from v_cfs;
```

修改视图的语法: `alter view 视图名称 as 修改后的查询语句;`

删除视图的语法: `drop view 视图名称; 例如 drop view v_cfs;`	

重命名视图的语法: `rename table 视图名称 to 新视图名称;`

## 事务

四大特性: 原子性、一致性、隔离性、持久性

事务语法:

```sql
begin;     -- 开始事务
your sql;  -- 编写sql
commit;    -- 提交事务, 若要放弃则使用 rollback; 
```

## 索引

定义:

- 单列索引: 一个索引只包含单个列, 一个表可以有多个单列索引, 但这不是组(联)合索引
- 组合索引: 又叫联合索引, 即一个索引包含多个列

建立: `create index 索引名 on 表名(列名1(长度),列名2(长度),...);`

> 如果列的数据类型是整型可以不写长度，如果是字符串就就写字符串的长度

查看: `show index from 表名;`

删除: `drop index 索引名 on 表名;`

好处: 提高查询效率

坏处: 

- 降低表的更新速度，因为当insert、update和delete的时候，mysql不仅要更新表的内容还要更新索引的内容
- 建立索引会占用磁盘空间的索引文件

索引使用示例:
	1. set profiling=1;  # 开启运行时间监测
	2. select * from areas where atitle='广州市';  # 没有建立索引的时候查询
	3. show profiles;  # 显示时间是0.00355700秒
	4. create index atitleindex on areas(atitle(20));  # 建立索引
	5. select * from areas where atitle='广州市';  # 建立索引后再次查询
	6. show profiles;  # 显示时间是0.00069875秒, 对比发现建立索引后查询速度提升
	7. drop index atitleindex on areas;  # 删除索引

## 扩展语句

查看时区

```sql
show variables like '%time_zone%';
```

查看时间

```sql
select now();
```

查看事件

```sql
show events;
```

删除事件

```sql
drop event event_name;
```

查看触发器

```sql
show triggers;
```

删除触发器

```sql
drop trigger trigger_name;
```

查看存储过程

```sql
show procedure status;
```

查看存储过程内容

```sql
show create procedure procedure_name;
```

删除存储过程

```sql
drop procedure procedure_name;
```

展示函数

```sql
show function status;
```

展示当前连接用户

```sql
show processlist;
```

> 踢掉客户端命令: kill id;

展示长连接最长时间

```sql
show global variables like "wait_timeout";
```

展示是否开启binlog

```sql
show variables like 'log_bin';
```

> binlog 记录了 DDL 和 DML 语句(除了数据查询语句select), 以事件形式记录
>
> 可以恢复 delete 和 truncate 删除的信息
>
> binlog 有3种记录方式: statement、row、mixed, statement记录执行的sql, row记录执行sql后的结果, mixed是前两者混合
>
> statement速度快占用磁盘空间小, 但在主从模式的情况下可能会出现数据不一致的问题, row正好相反, 一般建议使用row模式

## SQL命令

### 备份数据库

语法: `mysqldump -u 用户名 -p 数据库名 > ~/路径/备份文件.sql` , 例如:

```
mysqldump -u root -p students > ~/students_bak.sql
```

PS: 添加`--skip-extended-insert`可使导出数据中的每个INSERT语句都会独占一行, 方便查阅

### 还原数据库

语法: `mysql -u 用户名 -p 数据库名 < ~/路径/备份文件.sql;` , 例如:

```
mysql -u root -p students < ~/students_bak.sql
```

