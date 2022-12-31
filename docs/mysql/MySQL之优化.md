# MySQL之优化

## 排序优化总结

- MySQL支持filesort和index两种排序方式, Using index直接扫描索引本身完成排序, 效率高, Using filesort是文件排序, 效率慢

- order by满足使用 Using index 的条件:
  > - 符合索引最左前缀原则
  > - where语句后面条件和order by语句后面条件和在一起符合最左前缀原则
  
- 尽量使用索引排序, 即符合建立索引时的最左前缀原则

- order by 的条件不在索引列上, 会使用filesort排序, 索引尽量使用覆盖索引的列排序

- group by 与 order by 非常类似, 实质是先排序后分组, 按照索引最左前缀原则, 如果 group by 的结果不需要排序, 则添加 order by null 禁止排序, 也能提高查询效率

- where 条件高于 having, 尽量在 where 中限定条件而非在 having 中

> 扩展1: 
>
> Using index 是扫描二级索引或联合索引, 也就是非聚簇索引
>
> Using filesort 是扫描主键索引, 也就是聚簇索引
>
> 因为聚簇索引索引包含所有数据, 所以从磁盘读取ibd文件时加载的数据大, 效率不如非聚簇索引, 也就是说Using index 比 Using filesort 的效率高
>
> 
>
> 扩展2:
>
> 排序分为单路排序和双路排序
>
> 单路排序就是直接把select出来的结果集load到内存中, 然后排序, 排序之后的结果就是最终结果
>
> 双路排序仅仅是将 id 和排序的字段load到内存中(占用空间小), 然后排序, 排序完成后根据 id 再次回表拿到最终的结果
>
> 两者优点类似时间换空间, 空间换时间的意思
>
> MySQL通过系统系统变量 max_length_for_short_data(默认1048576字节, 1M)的大小确定使用哪种排序方式·
>
> 所有字段总长度小于 max_length_for_short_data 就使用单路排序, 否则使用双路排序, 比如表里字段比较多, 加起来超过1024字节, 就使用双路排序
>
> 
>
> 扩展3:
>
> MySQL会自动分配一块内存空间(sort_buffer), 如果排序数据量较小, 会在内存中也就是sort_buffer中完成, 否则会在磁盘中生成临时文件, 在读到内存中进行排序, 也就是说排序数据较大时, 查询效率降低查询时间增长
>
> 尽量使用limit限制结果集大小, 这样排序效率也会高

## 索引设计原则

- 建表之初不建索引, 后续根据实际业务, 把涉及到相关表的相关sql拿出来分析, 再根据分析结果建立索引, 比如建立联合索引时尽量覆盖代码中常用的覆盖条件(where后面的条件), 针对不同的情况还可建立一个或多个联合索引
- 对于差异较小的字段(即小基数字段), 不要建立索引, 意义不大, 比如gender, 不是0就是1, 无法体现二分查找的优势, 所以通过distinct查看字段的值占用总条数的比例, 高的建议建立索引, 低的不建议
- 尽量对占用空间较小字段类型设置索引, 例如 tinyint, 因为占用磁盘空间小, 查起来效率高
- 对于占用空间较大的字段, 例如vachar(255), 使用前几个字段建立索引, 类似 **key index(name(20), age, birthday)**, 但这样做的缺点是 order by 和 group by 该字段时无法使用索引
- where和order by冲突时, 优先针对where条件的字段建立索引(比如key index(name(20), 否则就是(key index(name))
- 对于慢sql针对性优化(默认定义查询时间超过10s的是慢sql, MySQL会记录这些sql到一个特定的文件中), 注意, 默认没有开启这项功能, 因为会影响性能, 如果是主从设计, 可以在备库开启
- 对于大概率使用范围作为条件的字段, 建立联合索引时要放到最后, 例如age, 那么就应该 key (name, gender, age), 因为where条件后字段使用 > 等范围条件后,  and 后面的字段便无法使用索引了

核心思路就是尽量利用一两个复杂的多字段联合索引, 覆盖80%的业务场景, 然后使用一两个辅助索引(联合索引或单字段索引)解决剩下的业务场景, 保证大数量级别的查询都能使用索引, 这样既可保证查询速度和性能不低

## Limit优化

### 示例1:

```sql
select * from a_table limit 10000, 10;
```

实际上MySQL查询了10010条数据, 然后舍弃前10000条, 只展示最后10条, 效率低下

优化:

```sql
select * from a_table where id > 10000 limit 10;
```

使用前提, **自增且连续**的主键排序分页查询

**假设中间删除了某些数据, 上述两条语句查询的结果会不一样, 原因就是 limit 本质依然是从第一条数据开始查询计算**

### 示例2:

```sql
select * from a_tble order by name limit 10000, 10;
```

假设 a_table 表中有联合索引(name, age, job), 理论上`order by name`会走索引, 但实际上不会, 原因就是前面多次提到的, MySQL 认为数据量较大, 使用联合索引扫描数据后还需要回表, 不如直接全表扫描

使用 Explain 可以看到 type 是 ALL, key 为空, extra 是 Using filesort, 证明没有使用索引

优化:

```sql
select * from a_table as a inner join (select id from a_table order by name limit 10000, 10) as b on a.id = b.id;
```

解释:

因为, `select id from a_table order by name limit 10000, 10` 不使用`select *`而是`select id`, 即使用了覆盖索引, 此时`order by name`走了索引, 使用 explain 可以看到 type 是 index, extra 是 Using index, 而且最终生成的这张临时表只有10条数据

而且,  `inner join`的条件是`on a.id = b.id`是主键关联, 其 type 是 eq_rf,  key 是 PRIMARY, 所以和上面的临时表联表查询的最终效率很高

## Join优化

先说两个算法

### 嵌套循环连接算法

即 Nested-Loop Join(NLJ) 算法

前提:

两张表 a 和 b, 数据结构一模一样, 其中 name 字段建立索引, a表一万条数据, b表100条数据

语句:

```sql
select * from a inner join b on a.name = b.name;
```

执行:

- 1、全表扫描 b 表, 拿到一条记录
- 2、拿到 1 中的结果, 通过 name 索引去 a 表中定位
- 3、重复步骤 1 和 2, 拿到最终结果

期间, b 表扫描100次, 由于使用了条件`on a.name = b.name`使用了索引, 也只扫描了100次

算法解释:

- MySQL的优化器一般会选择**小表做驱动表**, **大表做被驱动表**, 即 a 是被驱动表, b 是驱动表, 执行过程中先执行驱动表
- 对于 left join, 左表是驱动表, 右表是被驱动表
- 对于 right join, 正好和 left join 相反
- 小表的定义是**经过滤条件筛选后数据量小的表**是小表, 不是单纯的指整个表数据量小, 如果没有过滤条件那就指整个表

使用 explain 没有在结果的 extra 中看到 Using join buffer, 则表示当前 jon 语句使用了 NLJ 算法

如果被驱动表关联的字段没有索引, NLJ 算法效率低下, MySQL会选择 BNL 算法

### 基于块的嵌套循环链接算法

即 Block Nested-Loop Join(BNL) 算法

语句:

```sql
select * from a inner join b on a.age = b.age;
```

由于 age 字段没有索引, 所以不会使用 NLJ 算法, 而是 BNL算法

执行:

- 表小表 b, 即驱动表的数据放到 join_buffer 中
- 把大表 a, 即被驱动表中的每一行, 单独取出来和 join_buffer 中的数据比对
- 返回满足 join 条件的数据

期间, 两张表都是全表扫描, 共扫描 10000 + 100 = 10100 次, 同时 由于 join_buffer 表中的**数据在内存中且是无序的**, 从表 a 中取出来的数据最多要在表 b 中判断 100 次, 最后内存中判断的数量级是 10000 * 100 , 即百万级

PS: join_buffer 数据在内存中, 内存空间大小由参数 join_buffer_size 决定, 默认值好像是256K, 如果此值较小放不下 b 表数据, 则会分段放, 即先把 b 表的一部分数据放到 join_buffer 判断, 完成之后再放一部分数据, 直到放完, 无形中增加了计算时间

> 我猜 BNL 算法的中的 B, 即 block 就是指内存中这一块块的 join_buffer

> 现在说说为啥说 这种情况(即 on 的条件的字段没有索引)下, NLJ效率低, BNL效率高, 因为NLJ是扫描索引, 所索引文件在磁盘上, **NLJ是磁盘扫描, 而BNL出内存**
>
> 所以, 有索引的情况下一般用NLJ(效率高), 没索引的情况下一般分BNL(效率高)

### 优化

通过对NLJ和BNL的了解, 总结亮点优化原则:

- 关联字段加索引
- 小表驱动大表

扩展:

编写inner join语句时, 如果明确知道哪个表小, 可使用`straight_join`语法指定前面的表作为驱动表, 省去MySQL优化器判断的过程, 也可提高效率, 其语法格式为:

`小表 straight_join 大表`, 例如:

```sql
select * from b straight_join a on b.name = a.name;
```

**注意: straight_join 只适用于 inner join**(因为 left join 和 right join 已经指定了驱动表和被驱动表, 即执行顺序)

## IN和EXISTS优化

原则: 小表驱动大表, 小表的定义同join

1、当A表数据集大于B数据集时, IN 优于EXISTS, 例如:

a 表 100 万, b 表 100, 优先使用 in:

```sql
select * from a where id in (select id from b);
```

解释:

**in 的语法规则是先执行 in 后面的子查询, 再执行 in 前面的语句, 即先执行 b 后执行 a, 然后遍历 b 找到最终的结果集, 所以先执行的 b 是驱动表, 满足小表驱动大表**

2、当A表数据集小于B表数据集时, EXISTS优于IN, 例如:

a 表 100, b 表100 万, 优先使用 exists:

```sql
select * from a where exists (select 1 from b where b.id = a.id);
```

解释:

**exists 的语法规则是先执行 exists 前面的语句, 再执行后面的子查询, 即先执行 a 后执行 b, 然后将 a 放到 b 中验证条件, 所以先执行的 a 是驱动表, 满足小表驱动大表**

注意:

- exists(subquery) 只返回 True 或 False, 因此字查询中的 select * 可以用 select 1代替, 官方文档显示exists(subquery) 会忽略 select *, 因此没区别
- 如果 where 条件用的不是主键字段, 则该字段应建立索引提高查询速度
- exists 往往可以用 join 来代替, 何种方式最优需具体问题具体分析, 但是一般情况下尽量是用 join 少用 exists

## COUNT优化

先说一下各种count的效率

字段有索引

count(*) ≈ count(1) > count(字段) > count(主键)

字段没有索引

count(*) ≈ count(1) > count(主键) > count(字段)

注意: 针对MySQL5.6、5.7版本及以后

解释:

- MySQL对count(*)做了优化, 不取字段值, 按行累加, 效率最高

- count(1), 不取字段值, 使用常量1做统计(计算1的数量), 效率很高

- count(字段), 因为要取值(计算字段的数量), 所以比count(1)效率略低

- 实际上MySQL对count(主键)也做了优化, 当存在二级索引时, 优先使用二级索引, 否则就是主键索引

  因为主键索引是聚簇索引, 储存的数据量大, 相对储存部分数据的二级索引来讲, 扫描效率低一些

  这也就解释了为什么当字段存在索引时 count(字段) > count(主键) 

> 扩展: count(*)是SQL92定义的标准统计行数的语法, 跟数据库无关, 跟NUll和Not Null无关, 就是说`count(*)`**会统计值为Null的行**, `count(字段)`**不会统计Null值的行**

但是, 如果表的数据量很大, count依然会很慢

一些不太靠谱或方便的优化方式:

- 对于 myisam 引擎, 自动有一张表自动维护数量, 直接 select count(*) 就行
- 对于 innodb 引擎, 由于MVCC机制, 没有维护数量表, 只能硬查
- 如果对总数的精度要求不高, 只需要一个大约值, 可以使用 `show table status like 'a_table'`的方式查询, 这个性能很高
- 人工将总数维护到redis中, 每次对数据库进行插入或删除操作时对应的在redis中做incr和decr命令的操作, 缺点是很难保证一执行, 结果可能有出入
- 插入或删除数据的同时维护计数表, 且在同一个事务里操作

如果业务场景中 count 的次数比较多, 且表的数据量很大, 又有准确率的要求, 建议使用最后一个方式维护数量