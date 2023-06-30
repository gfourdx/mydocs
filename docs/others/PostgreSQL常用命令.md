# PostgreSQL常用命令

备份数据库

	pg_dump -h 127.0.0.1 -U postgres -d db >db.bak

还原数据库

```
psql -h 127.0.0.1 -U postgres -d db <  db.bak
```

创建数据库

```sql
create database testdb;
```

查看数据库

```
\l
```

删除数据库

```sql
drop database testdb;
```

连接数据库

```
\c testdb;
```

创建没有主键的表

```sql
create table tb1(id int not null,c_name varchar(32),c_phone char(14));
```

创建包含主键的表

```sql
-- 方法1
create table tb2(id int primary key,c_name varchar(32),c_phone char(14));
-- 方法2
create table tb3(id int not null,c_name varchar(32),c_phone char(14),primary key(id));
```

创建包含多个主键的表

```sql
create table tb4(id int,c_name varchar(32),c_phone char(14),primary key(id,c_name));
```

创建主键自动增长的表

```sql
-- 主键字段不用显示声明not null
create table tb5(id serial primary key, c_name varchar(32),c_phone char(14));
```

显示所有表

```
\d
```

显示某张表

```
\d tb1
```

删除表

```sql
drop table tb1;
drop table tb2,tb3;
```

扩展：模式(架构)，最常见的public就是一张模式(架构)

```sql
-- 创建模式(架构)：
create schema myschema;	

-- 删除模式(架构)：
drop schema myschema;
```

