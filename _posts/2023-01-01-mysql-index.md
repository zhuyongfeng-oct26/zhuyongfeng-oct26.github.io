---
layout: post
title: MySQL索引的分类和优化
categories: MySQL
description:
keywords: MySQL
---

# MySQL索引的分类和优化

![](/images/posts/mysql/05.png)

## 什么是索引

在MySQL中，索引（Index）是用于提高数据库查询性能的关键数据结构。它允许数据库系统更快地访问数据表中的特定数据，而无需扫描整个表。通过创建索引，数据库可以迅速定位到表中的特定行，从而极大地减少查询所需的时间和计算资源。

## 索引的分类

### 物理存储分类

从物理存储的角度来看，索引主要分为聚簇索引（即主键索引）和二级索引（也称为非聚簇索引或普通索引）。

- 主键索引按照表的主键构造一颗B+树，叶子节点中存放的就是整张表的行记录数据。基于主键索引的查询可以直接定位到完整的行记录数据，无需再回表查找。
- 二级索引构造的B+树，包含索引列和指向数据行的指针（叶子节点存放的是主键值）。基于非主键索引的查询需要多扫描一颗索引树，相对于聚簇索引的查询会慢一些。

在使用二级索引进行查找时，首先通过索引列定位到叶子节点中的主键值，然后再通过主键索引访问到数据行，这个过程被称为**回表**。

关于对 回表 的理解，我们举个例子，首先构造以下数据：

```sql
create table T
(
    id   int primary key,
    k    int not null,
    name varchar(16),
    index (k)
) engine = InnoDB;

INSERT INTO test.T (id, k, name) VALUES (100, 1, 'name_1');
INSERT INTO test.T (id, k, name) VALUES (200, 2, 'name_2');
INSERT INTO test.T (id, k, name) VALUES (300, 3, 'name_3');
INSERT INTO test.T (id, k, name) VALUES (400, 4, 'name_4');
INSERT INTO test.T (id, k, name) VALUES (500, 5, 'name_5');
INSERT INTO test.T (id, k, name) VALUES (600, 6, 'name_6');
```

```
mysql> select * from T;
+-----+---+--------+
| id  | k | name   |
+-----+---+--------+
| 100 | 1 | name_1 |
| 200 | 2 | name_2 |
| 300 | 3 | name_3 |
| 400 | 4 | name_4 |
| 500 | 5 | name_5 |
| 600 | 6 | name_6 |
+-----+---+--------+
6 rows in set (0.00 sec)
```

对应着主键索引和二级索引分别是：

![](/images/posts/mysql/06.png)

如果我们执行的是以下查询语句：

```sql
select * from T where k=5
```

- 首先，通过二级索引 k , 查询到ID值 500。
- 然后再到主键索引搜索一次，通过ID值 500 定位到叶子节点 R4，获取到完整的记录行数据，这个过程称为回表。

### 字段特性分类

从字段特性的角度来看，索引分为主键索引、唯一索引、普通索引、前缀索引。

**主键索引**

主键索引就是建立在主键字段上的索引，一张表最多只有一个主键索引，且索引列不允许有空值。

```sql
## 在创建表时指定主键
CREATE TABLE table_name (  
    column1 datatype,  
    column2 datatype,  
    column3 datatype,  
    ...  
    PRIMARY KEY (column1)  
);

## 在已经存在的表上添加主键
ALTER TABLE table_name ADD PRIMARY KEY (column_name);
```

**唯一索引**

一个表可以有多个唯一索引，但每个唯一索引的列组合必须是唯一的。此外，唯一索引允许有空值（NULL），但每个空值只能出现一次。

```sql
## 在创建表时指定唯一索引
CREATE TABLE table_name (  
    column1 datatype,  
    column2 datatype,  
    ...  
    UNIQUE (column1),  
    UNIQUE (column2, column3) -- 组合唯一索引  
);

## 在已经存在的表上添加唯一索引
ALTER TABLE table_name ADD UNIQUE (column1);
```

**普通索引**

普通索引（也称为单列索引或单值索引）是最基本的索引类型，它没有任何限制，只是用于提高查询性能。普通索引允许在索引列中有重复的值，并且也允许有空值（NULL）。

```sql
## 在创建表时指定普通索引
CREATE TABLE table_name (  
    column1 datatype,  
    column2 datatype,  
    ...  
    INDEX index_name (column1)  
);

## 在已经存在的表上添加普通索引
ALTER TABLE table_name ADD INDEX index_name (column1);
```

**前缀索引**

前缀索引是对字符串类型列的前几个字符创建索引。通过使用前缀索引，可以减少索引的大小，从而节省存储空间，并可能提高查询性能。

```sql
## 在创建表时指定前缀索引
CREATE TABLE table_name (  
    column1 VARCHAR(255),  
    ...  
    INDEX index_name (column1(prefix_length))  
);

## 在已经存在的表上添加前缀索引
ALTER TABLE table_name ADD INDEX index_name (column1(prefix_length));
```

选择合适的前缀长度是非常重要的，因为它会直接影响到索引的选择性和查询性能。如果前缀长度太短，可能会导致大量的重复值，从而降低索引的效果。如果前缀长度太长，则可能会接近全列索引的大小，失去前缀索引的优势。

可以通过查看字符串列的不同前缀长度的唯一值数量来帮助确定一个合适的前缀长度。例如使用以下查询来查看 `column1` 列前10个字符的唯一值数量：

```sql
SELECT COUNT(DISTINCT LEFT(column1, 10)) FROM table_name;
```

### 字段个数分类

从字段个数的角度来看，索引分为单列索引、联合索引。

**联合索引**

联合索引，也被称为复合索引或多列索引，是基于表中的两列或多列创建的索引。使用联合索引可以大大提高查询性能，尤其是当查询条件同时涉及多个列时。

```sql
## 创建联合索引
ALTER TABLE student ADD INDEX idx_name_age (name, age);
```

使用联合索引时，存在**最左匹配原则**，按照最左优先的方式进行索引的匹配，如果不遵循「最左匹配原则」，联合索引会失效，这样就无法利用到索引快速查询的特性了。

## 索引优化的方法

### **前缀索引优化**

前缀索引适用于那些需要索引长字符串列但又不需要完整字符串匹配的场景。

- 使用`COUNT(DISTINCT LEFT(column_name, n))`来测试不同前缀长度的唯一值数量，其中`n`是前缀长度，选择一个既能保持一定唯一性又不太长的前缀长度。

### **覆盖索引优化**

覆盖索引是指查询语句可以直接从索引中获取所需的数据，而无需回表到主表中进行二次查询的过程。使用覆盖索引可以减少IO操作和访问磁盘的次数，从而提高查询性能。

举个例子：

```sql
create table T
(
    id   int primary key,
    product_number    varchar(32) not null,
    product_price varchar(16),
    index (product_number, product_price)
) engine = InnoDB;

## 创建联合索引
ALTER TABLE T ADD INDEX idx_p_p (product_number, product_price);

......
select product_number,product_price from T where product_number = 'xxxx1';
```

以上SELECT语句中，我们通过 product_number 字段查询商品编码、价格信息，由于在联合索引（二级索引）的叶子节点中已经可以找到所需要 query 的字段数据，因此不需要再次检索主键索引，从而避免回表。