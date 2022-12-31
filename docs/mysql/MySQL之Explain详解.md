

# MySQL之Explain详解

---

## 创建表

```sql
CREATE TABLE `table1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL,
  `age` int(11) NOT NULL,
  `gender` bit(1) DEFAULT b'1',
  `birthday` datetime DEFAULT NULL,
  `location` varchar(64) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_name_age_gender` (`name`,`age`,`gender`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## 插入数据

```sql
INSERT INTO db1.table1(name, age, gender, birthday, location) VALUES
('aa', 20, b'1', '2001-1-1', 'beijing'),
('ab', 21, b'1', '2001-1-2', 'beijing'),
('ac', 22, b'1', '2001-1-3', 'beijing'),
('aa', 10, b'0', '2011-1-1', 'beijing'),
('ba', 30, b'1', '1991-1-1', 'tianjin'),
('ba', 30, b'1', '1991-12-12', 'shanghai'),
('ba', 40, b'0', '1981-6-6', 'chongqing'),
('ca', 20, b'1', '2001-1-1', 'shandong'),
('cb', 20, b'0', '2021-1-1', 'henan'),
('cd', 20, b'1', '2001-1-1', 'shanxi'),
('ce', 20, b'1', '2001-1-1', 'hebei'),
('de', 20, b'0', '1981-1-1', 'guangdong'),
('db', 20, b'0', '2001-1-1', 'taiwan');
```

## 测试

测试1 -- 全值匹配

```sql
explain select * from table1 where name='x';
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref   | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 130     | const | 1    | Using index condition |

name是varchar, 长度32, 4*32+2=130, 使用了索引 idx_name_age_gender 中的 name 字段

---

```sql
explain select * from table1 where name='x' and age=18;
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref         | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----------- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 134     | const,const | 1    | Using index condition |

同上, 同时 int 是 4 个字节, 所以130+4=134, 使用了索引 idx_name_age_gender 中的 name 和 age 字段

---

```sql
explain select * from table1 where name='x' and age=18 and gender=b'0';
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref               | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----------------- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 136     | const,const,const | 1    | Using index condition |

同上, bit 是 2 个字节,  所以 134+2=136, 使用了索引 idx_name_age_gender 中的 name、 age 和 gender 字段

> **注意: **
>
> **1、不同版本的MySQL计算key_lan的公式可能不一样, 不同编码的极端公式也可能不一样, 本例中用的是用的编码是utf8mb4(4个字节), 所以是计算公式是 4n*2, 如果用utf8编码就是3n+2**
>
> **2、上面3个输出中 rows 都是1, 是因为当前表中数据较少, 当数据量很大时, 上方三个输出中的 rows 会依次递减, 即使用联合索引时尽量全值(字段)匹配, 这样效率高**

---



测试2 -- 允许为空

```sql
ALTER TABLE db1.table1 MODIFY COLUMN age int(11) NULL;
explain select * from table1 where name='x' and age=18 and gender=b'0';
ALTER TABLE db1.table1 MODIFY COLUMN age int(11) NOT NULL;
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref               | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----------------- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 137     | const,const,const | 1    | Using index condition |

当设置 age 允许为空时, key_lan 变成 137, 多出来的一个字节就是用来记录 age 是否为空的

---



测试3 -- 字符串不加引号

```sql
explain select * from table1 where name=0;
```

| id   | select_type | table  | type | possible_keys       | key  | key_len | ref  | rows | Extra       |
| ---- | ----------- | ------ | ---- | ------------------- | ---- | ------- | ---- | ---- | ----------- |
| 1    | SIMPLE      | table1 | ALL  | idx_name_age_gender |      |         |      | 13   | Using where |

name 本身类型是 varchar, where 条件中使用时不添加引号, 就不会走索引



测试4 -- 顺序不一致

```sql
explain select * from table1 where age='xx' and name='x';
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref         | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----------- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 134     | const,const | 1    | Using index condition |

```sql
explain select * from table1 where age='xx' and name='x' and gender=b'0';
explain select * from table1 where gender=b'0' and age='xx' and name='x';
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref               | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----------------- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 136     | const,const,const | 1    | Using index condition |

> **MySQL内部会优化where条件后面的语句, 使其符合最左前缀原则, 从而可以使用索引**



测试5 -- 跳过索引字段

```sql
explain select * from table1 where name='xx' and gender=b'0';
```

| id   | select_type | table  | type | possible_keys       | key                 | key_len | ref   | rows | Extra                 |
| ---- | ----------- | ------ | ---- | ------------------- | ------------------- | ------- | ----- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | ref  | idx_name_age_gender | idx_name_age_gender | 130     | const | 1    | Using index condition |

使用 name 后跳过了 age 直接使用 gender, 未被最左前缀原则, 所以 key_len=130, 即只使用了索引中的 name 字段

---

```sql
explain select * from table1 where age=18 and gender=b'0';
```

| id   | select_type | table  | type | possible_keys | key  | key_len | ref  | rows | Extra       |
| ---- | ----------- | ------ | ---- | ------------- | ---- | ------- | ---- | ---- | ----------- |
| 1    | SIMPLE      | table1 | ALL  |               |      |         |      | 13   | Using where |

由于跳过了 name, 所以完全没有使用索引



测试5 -- 对索引做其他操作

```sql
explain select * from table1 where left(name, 4)='xx' ;
```

| id   | select_type | table  | type | possible_keys | key  | key_len | ref  | rows | Extra       |
| ---- | ----------- | ------ | ---- | ------------- | ---- | ------- | ---- | ---- | ----------- |
| 1    | SIMPLE      | table1 | ALL  |               |      |         |      | 13   | Using where |

由于对索引使用了函数, 所有没有使用索引

---

```sql
CREATE INDEX idx_birthday USING BTREE ON table1 (birthday);
explain select * from table1 where date(birthday)='2020-01-01';
explain select * from table1 where birthday>='2020-01-01 00:00:00' and birthday<='2020-01-01 23:59:59';
ALTER TABLE db1.table1 DROP INDEX idx_birthday;
```

| id   | select_type | table  | type | possible_keys | key  | key_len | ref  | rows | Extra       |
| ---- | ----------- | ------ | ---- | ------------- | ---- | ------- | ---- | ---- | ----------- |
| 1    | SIMPLE      | table1 | ALL  |               |      |         |      | 13   | Using where |

| id   | select_type | table  | type  | possible_keys | key          | key_len | ref  | rows | Extra                 |
| ---- | ----------- | ------ | ----- | ------------- | ------------ | ------- | ---- | ---- | --------------------- |
| 1    | SIMPLE      | table1 | range | idx_birthday  | idx_birthday | 6       |      | 1    | Using index condition |

由于对索引使用了函数, 所有没有使用索引; 但转换成比大小, 就使用了索引, 这就是索引优化

> **对索引进行计算、函数、手动或自动的类型转换都可能导致无法使用索引**



测试5 -- !=、<>、in、not、is null、is not null 条件

```sql
explain select * from table1 where name != 'aa';
explain select * from table1 where name <> 'aa';
explain select * from table1 where name not in ('aa', 'ab');
```

| id   | select_type | table  | type | possible_keys       | key  | key_len | ref  | rows | Extra       |
| ---- | ----------- | ------ | ---- | ------------------- | ---- | ------- | ---- | ---- | ----------- |
| 1    | SIMPLE      | table1 | ALL  | idx_name_age_gender |      |         |      | 13   | Using where |

```sql
explain select * from table1 where birthday  is null;
explain select * from table1 where birthday is not null;
```

| id   | select_type | table  | type | possible_keys | key  | key_len | ref  | rows | Extra       |
| ---- | ----------- | ------ | ---- | ------------- | ---- | ------- | ---- | ---- | ----------- |
| 1    | SIMPLE      | table1 | ALL  |               |      |         |      | 13   | Using where |

- 一般情况下这些都走不了索引
- 有时候不等于也能走索引
- in、or 有时候会走索引, 有时候不会走索引, MySQL内部优化器会根据检索比例、表大小等因素判断是否使用索引



**其他:**

- 使用 like 通配符, 只要第一个字符不是通配符就能走索引, 例如:

  - 'a%', 可以使用索引
  - '%a', 无法使用所有
  - '%a%', 无法使用所有
  - 'a%b', 可以使用索引

  > - **无论表里有多少条数据, like ‘a%’ 都会走所有, in 或者 or 在表大的时候走索引, 表小的时候不走索引**
  >
  > - name > 'a' and age > 20 不会走索引 (个人猜测使用 > 导致结果集范围比较大, 所以默认不使用索引)
  > - name like 'a%' and age > 20 就会走索引
  > - 之所以这样, 是因为从MySQL5.6版本开始引入了**索引下推原则**(适用于like查询): 在遍历索引的过程中优先判断索引包含的字段, 过滤掉不符合要求的记录后再回表, 可减少回表次数, 提高查询效率
  > - 简单说就是 like 完之后继续匹配 age 使范围缩小, 而**联合索引中第一个字段**使用 > **不一定**会使用索引(估计和结果集大小有关, like相对来说比较小), 也不会使用索引下推
  > - 索引下推会减少回表次数, 仅限于使用innodb的二级索引, 因为主键索引是聚簇索引, 叶子节点包含所有数据, 索引下推不会起到减少查询全行数据的效果

- 对索引使用范围条件是, 根据条件的大小, 可能会有不同的结果, 例如:

  - where 100 < speed < 100000000 就有可能不走索引
  - where 100 < speed < 200 就有可能走索引

  这种情况就好比, 表中的数据量少, 如果使用非主键索引, 还要回表, 不如直接全表扫描来的快
