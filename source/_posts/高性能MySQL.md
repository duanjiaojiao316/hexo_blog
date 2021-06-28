---
title: 高性能MySQL
categories:
  - 数据库
translate_title: highperformance-mysql
date: 2020-11-29 22:31:44
---
# 高性能MySQL

## 高性能索引

使用ORM对象关系映射工具也需要关注索引，关于ORM工具，不论多复杂都不能完成完美的查询优化。

### B-Tree索引

所有的值按顺序存储，每个叶子页到根的距离相同，避免了全表扫描搜索。从索引的根节点开始进行搜索，根节点的槽存放指向子节点的指针，存储引擎根据这些指针向下层查询。通过子节点的值和要查询数据的值进行比较找到合适的指针指向下一层子节点。这些指针定义了子节点的值的上限和下限。叶子节点的指针指向的是被索引的数据，不是其他节点页。

B-Tree对索引列是顺序组织存储的，适合查找范围数据。

在定义B-Tree索引时，索引列多于一个，索引列的顺序与`key(lastname,firstname,dob)`中的顺序无关，依据的顺序是`CREATE TABLE`语句中定义索引时列的顺序。

#### 查询类型

- 全值匹配：查询姓名为xxx xxx，出生日期为yyyy-mm-dd的人
- 匹配最左前缀： 查询姓为xxx的人
- 匹配列前缀： 查询姓以J开头的人
- 匹配范围值： 查询姓在某两个值之间的人
- 精确匹配某一列并范围匹配某一列
- 只访问索引的查询，无须访问数据行

#### 查询限制

- 如果不是最左列开始查询，则无法使用索引
- 不能跳过索引的列
- 如果某一列是范围查询右边的所有列无法使用索引优化查找

### 哈希索引

精确匹配所有索引列的查询才有效，存储引擎会根据所有的所有列计算一个哈希值。只有Memory显式支持哈希索引，并且是非唯一索引，如果哈希值相同，会以链表的形式存放多个数据记录在同一个哈希条目中。

#### 查询限制

- 只包含哈希值和指针，不能避免读取数据行
- 不是按照顺序存储的，无法实现排序
- 不支持用于部分索引列的匹配查找，哈希值是通过所有索引列计算哈希值的
- 只支持等值比较查找 = ,IN(),<>,<=>
- 访问哈希索引的数据非常快，除非出现哈希冲突，存储引擎必须遍历所有的行指针
- 出现哈希冲突较多，索引维护的代价会很高

#### InooDB的自适应哈希索引

在B-Tree的基础上添加哈希索引。用户无法操纵，但是可以选择开启和关闭。

#### 使用什么计算哈希值

- CRC32，
- SHA1()，MD5()作为哈希函数产生的哈希值很长
- 自定义哈希函数
- 或者使用SHA1()，MD5()作为哈希函数结果的一部分来作为哈希值（性能低，实现简单）
- FNV64函数以插件方式使用，哈希值是64位的，冲突比CRC32少很多

### 空间数据索引（R-Tree）

从所有维度索引数据，可以使用任何维度来组合查询。

MySQL的GIS支持不完善很少使用

PostgreSQL的PostGIS比较好

### 全文索引

查找文本的关键词，类似搜索引擎的工作。

## SQL优化

**负向查询不能使用索引**

```sql
select name from user where id not in (1,3,4);
```

应该修改为:

```
select name from user where id in (2,5,6);
```

**前导模糊查询不能使用索引**

```sql
select name from user where name like '%zhangsan'
```

非前导则可以:

```sql
select name from user where name like 'zhangsan%'
```

建议可以考虑使用 `Lucene` 等全文索引工具来代替频繁的模糊查询。

**数据区分不明显的不建议创建索引**

如 user 表中的性别字段，可以明显区分的才建议创建索引，如身份证等字段。

**字段的默认值不要为** null

这样会带来和预期不一致的查询结果。

**在字段上进行计算不能命中索引**

```sql
select name from user where FROM_UNIXTIME(create_time) < CURDATE();
```

应该修改为:

```sql
select name from user where create_time < FROM_UNIXTIME(CURDATE());
```

**最左前缀问题**

如果给 user 表中的 username pwd 字段创建了复合索引那么使用以下SQL 都是可以命中索引:

```sql
select username from user where username='zhangsan' and pwd ='axsedf1sd'

select username from user where pwd ='axsedf1sd' and username='zhangsan'

select username from user where username='zhangsan'
```

但是使用

```sql
select username from user where pwd ='axsedf1sd'
```

是不能命中索引的。

**如果明确知道只有一条记录返回**

```sql
select name from user where username='zhangsan' limit 1
```

可以提高效率，可以让数据库停止游标移动。

**不要让数据库帮我们做强制类型转换**

```sql
select name from user where telno=18722222222
```

这样虽然可以查出数据，但是会导致全表扫描。需要修改为

```sql
select name from user where telno='18722222222'
```

**如果需要进行 join 的字段两表的字段类型要相同**

不然也不会命中索引