title: MySQL-高级
author: 顾梦
tags:
  - Mysql
categories:
  - 学习笔记
index_img: 'https://cdn.jishuqin.cn//202304192055832.png'
date: 2023-07-09 16:47:40
---
# MySQL-高级

<p class="note note-success">
  此文是我学习mysql时笔记，教程链接：<a href="https://www.bilibili.com/video/BV1Kr4y1i7ru" target="_blank">https://www.bilibili.com/video/BV1Kr4y1i7ru</a>
</p>

## 一、存储引擎

### 1.MySQL体系结构

 ![](https://cdn.jishuqin.cn//MySQL-%E9%AB%98%E7%BA%A7/image-20230630192648123.png)

- 连接层

  最上层是一些客户端和链接服务，主要完成一些类似于连接处理、授权认证、及相关的安全方案。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

- 服务层

  第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。

- 引擎层

  存储引擎真正的负责了M小ySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。

- 存储层

  主要是将数据存储在文件系统之上，并完成与存储引擎的交互

### 2.存储引擎简介

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型。

1. 在创建表时，指定存储引擎

   ```mysql
   CREATE TABLE 表名{
   	字段1 字段1类型 [COMMENT 字段1注释],
   	......
   	字段n 字段n类型 [COMMENT 字段n注释]
   }ENGINE=INNODB [COMMENT 表注释]；
   ```

2. 查看当前数据库支持的存储引擎

   ```mysql
   SHOW ENGINES;
   ```

![](https://cdn.jishuqin.cn//MySQL-%E9%AB%98%E7%BA%A7/image-20230630193726848.png)

### 3.存储引擎特点

#### InnoDB

- 介绍：InnoDB是一种兼顾高可靠性和高性能的通用存储引擎，在MySQL 5.5之后，InnoDB 是默认的 MySQL 存储引擎。

- 特点：

  1. DML操作遵循ACID模型，支持**事务**；
  2. **行级锁**，提高并发访问性能：
  3. 支持**外键** FOREIGN KEY 约束，保证数据的完整性和正确性；

- 文件

  xxx.ibd：xxx代表的是表名，innoDB 引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引。
  参数：innodb_file_per_table

  ![](https://cdn.jishuqin.cn//MySQL-%E9%AB%98%E7%BA%A7/image-20230630200019249.png)

#### MyISAM

- 介绍：MyISAM是MySQL早期的默认存储引擎。

- 特点：

  1. 不支持事务，不支持外键
  2. 支持表锁，不支持行锁
  3. 访问速度快

- 文件：

  xxx.sdi：存储表结构信息
  xxx.MYD：存储数据
  xxx.MYI：存储索引

#### Memory

- 介绍：Memory 引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。
- 特点：
  1. 内存存放
  2. hash 索引（默认）
- 文件：xxx.sdi：存储表结构信息

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230630200854814-16883049088341.png)

**面试题：InnoDB与MyISAM的区别？**

InnoDB支持事务安全、行级锁和外键，MyISAM不支持。

### 4.存储引擎选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

- <u>InnoDB</u>：是Mysql的默认存储引擎，支持事务、外键。如果应用<u>对事务的完整性有比较高的要求</u>，在并发条件下<u>要求数据的一致性</u>，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择。
- MyISAM：如果应用是以<u>读操作和插入操作为主</u>，只有<u>很少的更新和删除操作</u>，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的。
- MEMORY：将所有数据保存在内存中，<u>访问速度快</u>，通常用于<u>临时表及缓存</u>。EMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。

## 二、索引 ⭐

### 1.索引概述

- 
  介绍：索引(index)是帮助 MySQL <u>高效获取数据</u>的<u>数据结构</u>（有序）。

- 优缺点：

  | 优势                                                         | 劣势                                                         |
  | ------------------------------------------------------------ | :----------------------------------------------------------- |
  | 提高数据检索的效率，降低数据库的 IO 成本                     | 索引列也是要占用空间的。                                     |
  | 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗。 | 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低。 |

### 2.索引结构

MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的结构，主要包含以下几种：

| 索引结构              | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| **B+Tree 索引**       | **最常见的索引类型，大部分引擎都支持B+树索引**               |
| Hash 索引             | 底层数据结构是用哈希表实现的，只有精确匹配索引列的查询才有效，不支持范围查询 |
| R-tree（空间索引）    | 空间索引是 MyISAM 引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-text（全文索引） | 是一种通过建立倒排索引，快速匹配文档的方式。类似于Lucene,Solr,ES |

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230702215652288.png)

**我们平常所说的索引，如果没有特别指明，都是指B+树结构组织的索引。**

- B+Tree

  MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree,提高区间访问的性能。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230702223006023.png)

- Hash

  哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在hash表中。

  如果两个（或多个）键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞)，可以通过链表来解决。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230702224041782.png)

  Hash索引特点：

  - Hash 索引只能用于对等比较(=，in),不支持范围查询(between，>，<，...)
  - 无法利用索引完成排操作
  - 查询效率高，通常只需要一次检索就可以了，效率通常要高于 B+tree 索引

  存储引擎支持：在MySQL中，支持hash索引的是Memory引擎，而InnoDB中具有自适应hash中能，hash索引是存储引擎根据B+Tree索引在指定条件下自动构建的。

**面试题：为什么InnoDB存储引擎选择使用 B+tree 索引结构？**

- 相对于二叉树，层级更少，搜索效率高；
- 对于 B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
- 相对Hash索引，B+tree支持范围匹配及排序操作；

### 3.索引分类

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230702225311526.png)

在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种：

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230702225742717.png)

聚集索引选取规则：

- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则 InnoDB 会自动生成一个 rowid 作为隐藏的聚集索引。

**思考：**

1. 以下 SQL 语句，哪个执行效率高？为什么？

   ```mysql
   select * from user where id=10;
   
   select * from user where name='Arm';
   
   备注：id为主键，name字段创建的有索引;
   ```

   根据id查询比根据name查询效率高，根据name查询需要扫描两个索引效率低。

2. InnoDB主键索引的B+tree高度为多高呢？

   ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230702231721965.png)

### 4.索引语法

- 创建索引

  ```mysql
  CREATE [UNIQUE|FULLTEXT] INDEX index_name ON table_name (index_col_name,...);	
  ```

- 查看索引

  ```mysql
  SHOW INDEX FROM table_name;
  ```

- 删除索引

  ```mysql
  DROP INDEX index_name ON table_name;
  ```

### 5.SQL 性能分析

- SQL 执行频率

  MySQL 客户端连接成功后，通过shoW[session|global] status命令可以提供服务器状态信息。通过如下指令，可以查看当前数据库的 INSERT、UPDATE、DELETE、SELECT 的访问频次：

  ```mysql
  SHOW GLOBAL STATUS LIKE 'Com_______';	
  ```

- 慢查询日志

  慢查询日志记录了所有执行时间超过指定参数(long_query_.time,单位：秒，默认10秒)的所有SQL语句的日志。
  MySQL 的慢查询日志默认没有开启，需要在 MySQL 的配置文件（/etc/my.cnf）中配置如下信息：

  ```bash
  # 开启MySQL慢日志查询开关
  slow_query_log=1
  # 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
  long_query_time=2
  ```

- profile 详情

  执行一系列的业务SQL的操作，然后通过如下指令查看指令的执行耗时：

  ```mysql
  #查看每一条SQL的耗时基本情况
  show profiles;
  
  #查看指定query id的SQL语句各个阶段的耗时情况
  show profile for query query_id;
  
  #查看指定query_id的SQL语句CPU的使用情况
  show profile cpu for query query_id;
  ```

- explain执行计划

  EXPLAIN 执行计划各字段含义：

  - id

    select查询的序列号，表示查询中执行select子句或者是操作表的顺序(id相同，执行顺序从上到下；id不同，值越大，越先执行)。

  - select_type

    表示 SELECT 的类型，常见的取值有SIMPLE(简单表，即不使用表连接或者子查询)、PRIMARY(主查询，即外层的查询)、UNION(UNION中的第二个或者后面的查询语句)、SUBQUERY(SELECT/WHERE之后包含了子查询)等

  - **type**

    表示连接类型，性能由好到差的连接类型为 NULL、system、const、eq_ref、ref、range、index、all。

  - **possible_key**

    显示可能应用在这张表上的索引，一个或多个。

  - **key**

    实际使用的索引，如果为NULL,则没有使用索引。

  - **key_len**

    表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好。

  - rows

    MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值，可能并不总是准确的。

  - filtered

    表示返回结果的行数占需读取行数的百分比，filtered 的值越大越好。
  
### 6.索引使用

- 最左前缀法则

  如果索引了多列（联合索引），要遵守最左前缀法测。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。
  如果跳跃某一列，<u>索引将部分失效（后面的字段索引失效）。</u>

- 范围查询

  联合索引中，出现范围查询(>，<)，<u>范围查询右侧的列索引失效</u>

- 索引列运算

  不要在索引列上进行运算操作，索引将失效。

- 字符串

  字符串类型字段使用时，不加引号，索引将失效。

- 模糊查询

  如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

- or连接的条件

  用or分割开的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

- 数据分布影响

  如果MySQL评估使用索引比全表更慢，则不使用索引。

- SQL 提示

  SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

  use index:

  ```mysql
  explain select*from tb_user use index(idx_user_pro)where profession='软件工程'；
  ```

  ignore index:

  ```mysql
  explain select*from tb_user ignore index(idx_user_pro)where profession='软件工程'；
  ```

  force index:

  ```mysql
  explain select*from tb_user force index(idx_user_pro)where profession='软件工程'；
  ```

- 覆盖索引

  尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少select*。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230704235902137.png)
  
- 前缀索引

  当字段类型为字符串（varchar,text等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO,影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

  语法：

  ```mysql
  creae index idx_xxxx on table_name(column(n));
  ```

  前缀长度： 

  可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

  ```mysql
  select count(distinct email)/count(*) from tb_user;
  select count(distinct substring(email,1,5))/count(*) from tb_user;	 
  ```

- 单列索引与联合索引

  单列索引：即一个索引只包含单个列。

  联合索引：即一个索引包含了多个列。

  在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。

### 7.索引设计原则

1. 针对于数据量较大，且查询比较频繁的表建索引。
2. 针对于常作为查询条件(where)、排序(order by)、分组(group by)操作的字段建立索引。
3. 尽量选择区分度高的列作为索引，尽量建立<u>唯一索引</u>，区分度越高，使用索引的效率越高。
4. 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立<u>前缀索引</u>。
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以<u>覆盖索引</u>，节省存储空间，避免回表，提高查询效率。
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7. 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。

## 三、SQL优化

### 1.插入数据

- insert 优化

  1. 批量插入

  2. 手动提交事务

  3. 主键顺序插入

- 大批量插入数据

  如果一次性需要插入大批量数据，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令进行插入。操作如下：
  
  ```mysql
  #客户端连接服务端时，加上参数 --local-infile
  mysql --local-infile -u root -p
  #设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
  set global local_infile=1;
  #执行load指令将准备好的数据，加载到表结构中
  load data local infile '/root/sql1.log' into table `tb_user` fields terminated by ',' lines terminated by '\n';
  ```

### 2.主键优化

- 数据组织方式

  在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为**索引组织表**(index organized table **IOT**)。

- 页分裂

  页可以为空，也可以填充一半，也可以填充100%。每个页包含了2-N行数据（如果一行数据多大，会行溢出），根据主键排列。

  ![主键顺序插入](https://cdn.jishuqin.cn//MySQL-高级/image-20230705150818074.png)

  页分裂：

  ![主键乱序插入](https://cdn.jishuqin.cn//MySQL-高级/image-20230705151047951.png)

  页合并：

  当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记(flaged)为删除并且它的空间变得允许被其他记录声明使用。

  当页中删除的记录达到MERGE THRESHOLD(默认为页的50%)，InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。

  合并前：

  ![合并前](https://cdn.jishuqin.cn//MySQL-高级/image-20230705152251245.png)

  合并后：

  ![合并后](https://cdn.jishuqin.cn//MySQL-高级/image-20230705152335931.png)

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230705152408686.png)

- 主键设计原则

  1. 满足业务需求的情况下，尽量降低主键的长度。 
  2. 插入数据时，尽量选择顺序插入，选择使用AUTO INCREMENT自增主键。
  3. 尽量不要使用UUD做主键或者是其他自然主键，如身你证号。
  4. 业务操作时，避免对主键的修改。

### 3.order by优化

1. Using filesort：通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。
2. Using index：通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

总结:

- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
- 尽量使用覆盖索引。
- 多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则(ASC/DESC)。
- 如果不可避免的出现 filesort,大数据量排序时，可以适当增大排序缓冲区大小sort buffer size(默认256k)。

### 4.group by优化

```mysql
#删除掉目前的联合索引idx_user_pro_age_sta
drop index idx_user_pro_age_sta on tb_user;
#执行分组操作，根据orofession字段分组
explain select profession,count(*) from tb_user group by profession;
#创建索引
Create index idx_user_pro_age_sta on tb_user(profession,age,status);
#执行分组操作，根据profession字段分组
explain select profession,count(*) from tb_user group by profession;
#执行分组操作，根据profession字段分组
explain select profession,count(*) from tb_user group by profession,age;
```

总结:

- 在分组操作时，可以通过索引来提高效率
- 分组操作时，索引的使用也是满足最左前缀法则的。

### 5.limit优化

一个常见又非常头疼的问题就是limit2000000,10,此时需要MySQL排序前2000010记录，仅仅返回2000000-2000010的记录，其他记录丢弃，查询排序的代价非常大。

优化思路：一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。

```mysql
explain select * from tb_sku t,(select id from tb_sku order by id limit 2000000,10) a where t.id = a.id;
```

### 6.count优化

```mysql
explain select count(*) from tb_user;
```



- MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行count(*)的时候会直接返回这个数，效率很高；
- InnoDB 引擎就麻烦了，它执行count(*)的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。

优化思路：自己计数。

- count 的几种用法

  1. count(主键)：InnoDB 引擎会遍历整张表，把每一行的主键  id 值都取出来，返回给服务层。服务层拿到主键后，直接按行进行累加（主键不可能为null)。

  2. count(字段)：

     没有 not null 约束：InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为 null，不为 null，计数累加。
     有not null 约束：InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加。

  3. count(1)：InnoDB 引擎遍历整张表，但不取值。服务层对于返回的每一行，放一个数字“1”进去，直接按行进行累加。

  4. count(*)：InnoDB 引擎并不会把全部字段取出来，而是专门做了优化，不取值，服务层直接按行进行累加。

  按照效率排序：count(字段) < count(主键) < count(1) ≈ `count(*)`，所以尽量使用`count(*)`。

### 7.update优化

```mysql
update student set no='2000100100' where id =1;

update student set no='2000100105' where name='韦一笑'；
```

InnoDB 的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则会从行锁升级为表锁。

## 四、视图/存储过程/触发器

### 1.视图

- 介绍

  视图(Viw)是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

- 创建

  ```mysql
  CREATE [OR REPLACE] VIEW 视图名称(列名列表)] AS SELECT语句 [WITH[CASCADED | LOCAL] CHECK OPTION}]
  ```

- 查询

  ```mysql
  -- 查看创建视图语句
  SHOW CREATE VIEW 视图名称;
  
  -- 查看视图数据
  SELECT * FROM 视图名称......;
  ```

- 修改

  ```mysql
  -- 方式1
  CREATE [OR REPLACE] VIEW 视图名称(列名列表)] AS SELECT语句 [WITH[CASCADED | LOCAL]CHECK OPTION]
  -- 方式2
  ALTER VIEW 视图名称[(列名列表)] AS SELECT语句 [WITH [CASCADED | LOCAL] CHECK OPTION]
  ```

- 删除

  ```mysql
  DROP VIEW [IF EXISTS] 视图名称 [,视图名称] ...
  ```

- 视图的检查选项

  当使用 WITH CHECK OPTION子句创建视图时，MySQL会通过视图检查正在更改的每个行，例如插入，更新，删除，以使其符合视图的定义。MySQL 允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性。为了确定检查的范围，mysql 提供了两个选项：CASCADED 和 LOCAL,默认值为 CASCADED。

- 视图的更新

  要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项，则该视图不可更新：

  1. 聚合函数或窗口函数（SUM()、MIN()、MAX()、COUNT()等）
  2. DISTINCT
  3. GROUP BY
  4. HAVING
  5. UNION 或者 UNION ALL

- 作用

  1. 简单

     视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的每次指定全部的条件。

  2. 安全

     数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据

  3. 数据独立
     视图可帮助用户屏蔽真实表结构变化带来的影响。

### 2.存储过程 

- 介绍

  存储过程是事先经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。 

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230707092338699.png)

- 特点

  1. 封装，复用

  2. 可以接收参数，也可以返回数据

  3. 减少网络交互，效率提升

- 创建

  ```mysql
  CREATE PROCEDURE 存储过程名称([参数列表])
  BEGIN
  		--SQL语句
  END;		
  ```

- 调用

  ```mysql
  CALL 名称([参数]);
  ```

- 查看

  ```mysql
  -- 查询指定数据库的存储过程及状态信息
  SELECT * FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA='xxx';
  -- 查询某个存储过程的定义	
  SHOW CREATE PROCEDURE 存储过程名称；
  ```

- 删除

  ```mysql
  DROP PROCEDURE [IF EXISTS] 存储过程名称;
  ```

  注意：在命令行中，执行创建存储过程的SQL时，需要通过关键字delimiter指定SQL语句的结束符。

- 变量

  1. 系统变量：MySQL服务器提供，不是用户定义的，属于服务器层面。分为全局变量(**GLOBAL**)、会话变量(**SESSION**)。

     查看系统变量

     ```mysql
     SHOW [SESSION|GLOBAL] VARIABLES; -- 查看所有系统变量
     SHOW [SESSION|GLOBAL] VARIABLES LIKE '......'; -- 可以通过LIKE模糊匹配方式查找变量
     SELECT @@[SESSION|GLOBAL] 系统变量名; -- 查看指定变量的值
     ```

     设置系统变量

     ```mysql
     SET [SESSION|GLOBAL] 系统变量名=值;
     SET @@[SESSION|GLOBAL] 系统变量名=值;
     ```

     注意：

     ​	如果没有指定SESSION/GLOBAL,默认是SESSION,会话变量。
     ​	mysql服务重新启动之后，所设置的全局参数会失效，要想不失效，可以在/etc/my.cnf由配置。

  2. 用户自定义变量：用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用“@变量名”使用就可以。其作用域为当前连接。

     赋值：

     ```mysql
     SET @var_name = expr [,@var_name expr]...;
     SET @var_name := expr [,@var_name :=expr]...;
     
     SELECT @var_name:=expr [,@var_name :=expr]...;
     SELECT 字段名 INTO @var_name FROM 表名；
     ```

     使用：

     ```mysql
     SELECT @var_name;
     ```

     注意：用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NULL。

  3. 局部变量：根据需要定义的在局部生效的变量，访问之前，需要DECLARE声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的BEGN ... END块。

     声明：

     ```mysql
     DECLARE 变量名 变量类型[DEFAULT ...];
     ```

     变量类型就是数据库字段类型：INT、BIGINT、CHAR、VARCHAR、DATE、TIME等。

     赋值：

     ```mysql
     SET 变量名=值；
     SET 变量名:=值;
     SELECT 字段名 INTO 变量名 FROM 表名 ...;
     ```

- if

  语法：

  ```mysql
  IF 条件1 THEN
  	...
  ELSEIF 条件2 THEN		-- 可选
  	...
  ELSE				-- 可选
  	...
  END IF;
  ```

- 参数

  | 类型  | 含义                                         | 备注 |
  | :---: | :------------------------------------------- | ---- |
  |  IN   | 该类参数作为输入，也就是需要调用时传入值     | 默认 |
  |  OUT  | 该类参数作为输出，也就是该参数可以作为返回值 |      |
  | INOUT | 既可以作为输入参数，也可以作为输出参数       |      |

  用法：

  ```mysql
  CREATE PROCEDURE 存储过程名称([IN/OUT/INOUT 参数名 参数类型])
  BEGIN
  	-- SQL语句
  END;
  ```

- case
  
  语法一：
  
  ```mysql
  CASE case_value
  	WHEN when_value1 THEN statement_list1
  	[WHEN when_value2 THEN statement_list2]...
  	[ELSE statement_list]
  END CASE;
  ```
  
  语法二：
  
  ```mysql
  CASE
  	WHEN search_condition1 THEN statement_list1
  	[WHEN search_condition2 THEN statement_list2]...
  	[ELSE statement_list]
  END CASE;
  ```
  
- while

  whil循环是有条件的循环控制语句。满足条件后，再执行循环体中的SQL语句。具体语法为：

  ```mysql
  # 先判断条件，如果条件为true，则执行逻辑，否则，不执行逻辑
  WHILE 条件 DO
  	SQL逻辑...
  END WHILE;
  ```

- repeat

  repeat 是有条件的循环控制语句，当满足条件的时候退出循环。

  具体语法为：

  ```mysql
  #先执行一次逻辑，然后判定逻辑是否满足，如果满足，则退出。如果不满足，则继续下一次循环
  REPEAT
  	SQL 逻辑...
  	UNTIL 条件
  END REPEAT;
  ```

- loop

  LOOP实现简单的循环，如果不在SQL逻辑中增加退出循环的条件，可以用其来实现简单的死循环。LOOP可以配合一下两个语句使用：

  - LEAVE：配合循环使用，退出循环。
  - ITERATE：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

  ```mysql
  [begin_label:] LOOP
  	SQL 逻辑...
  END LOOP [end_label];
  
  LEAVE label; -- 退出指定标记的循环体
  ITERATE label; -- 直接进入下一次循环
  ```

- 游标

   **游标（CURSOR）** 是用来存储查询结果集的数据类型，在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用包括游标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

  声明游标：

  ```mysql
  DECLARE 游标名称 CURSOR FOR 查询语句；
  ```

  打开游标:

  ```mysql
  OPEN 游标名称;
  ```

  获取游标记录：

  ```mysql
  FETCH 游标名称 INTO 变量[,变量];
  ```

  关闭游标：

  ```mysql
  CLOSE 游标名称;
  ```

- 条件处理程序

  **条件处理程序（Handler）** 可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。

  具体语法为：

  ```mysql
  DECLARE handler_action HANDLER FOR condition_value [,condition_value]... statement;
  
  handler_action
  	CONTINUE:继续执行当前程序
  	EXIT:终止执行当前程序
  	
  condition_value
  	SQLSTATE sqlstate_value:状态码，如 02000
  	SQLWARNING:所有以01开头的SQLSTATE代码的简写
  	NOT FOUND:所有以02开头的SQLSTATE代码的简写
  	SQLEXCEPTION:所有没有被SQLWARNING 或 NOT FOUND捕获的SQLSTATE代码的简写
  ```

### 3.存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是 IN 类型的。

具体语法如下：

```mysql
CREATE FUNCTION 存储函数名称([参数列表])
RETURNS type [characteristic ...]
BEGIN
	-- SQL语句
	RETURN ...;
END;
```

characteristic 说明:

- DETERMINISTIC：相同的输入参数总是产生相同的结果。
- NO SOL：不包含SQL语句。
- READS SQL DATA：包含读取数据的语句，但不包含写入数据的语句。

### 4.触发器

- 介绍

  触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。
  使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

  | 触发器类型      | NEW 和 OLD                                             |
  | --------------- | ------------------------------------------------------ |
  | INSERT 型触发器 | NEW 表示将要或者已经新增的数据                         |
  | UPDATE 型触发器 | OLD 表示修改之前的数据，NEW 表示将要或已经修改后的数据 |
  | DELETE 型触发器 | OLD 表示将要或者已经删除的数据                         |

- 语法

  创建：

  ```mysql
  CREATE TRIGGER trigger_name
  BEFORE/AFTER INSERT/UPDATE/DELETE
  ON tbl_name FOR EACH ROW -- 行级触发器
  BEGIN
  	trigger_stmt;
  END;
  ```

  查看：

  ```mysql
  SHOW TRIGGERS;
  ```

  删除：

  ```mysql
  DROP TRIGGER [schema_name] trigger_name;-- 如果没有指定 schema_name，默认为当前数据库。
  ```

## 五、锁

### 1.概述

- 介绍

  锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

- 分类

  MySQL 中的锁，按照锁的粒度分，分为以下三类：

  1. 全局锁：锁定数据库中的所有表。
  2. 表级锁：每次操作锁住整张表。
  3. 行级锁：每次操作锁住对应的行数据。

### 2.全局锁

- 介绍

  全局锁就是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的DML的写语句，DDL语句，已经更新操作的事务提交语句都将被阻塞。
  其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230707212941097.png)

- 特点

  数据库中加全局锁，是一个比较重的操作，存在以下问题：

  1. 如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆。
  2. 如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志（binlog）,会导致主从延迟。

  

  在InnoDB引擎中，我们可以在备份时加上参数 --single-transaction 参数来完成不加锁的一致性数据备份。
  ```mysql
  mysqldump --single-transaction -uroot -p123456 itcast>itcast.sql
  ```

### 3.表级锁

- 介绍

  表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中。

  主要分为以下三类：

  1. 表锁
  2. 元数据锁（meta data lock，MDL）
  3. 意向锁

- 表锁

  对于表锁，分为两类：

  1. 表共享读锁（read lock)
  2. 表独占写锁（write lock）

  语法：

  1. 加锁：lock tables 表名...  read/write。
  2. 释放锁：unlock tables /客户端断开连接。

  ![读锁](https://cdn.jishuqin.cn//MySQL-高级/image-20230707213906222.png)

  ![写锁](https://cdn.jishuqin.cn//MySQL-高级/image-20230707214548831.png)

  

  读锁不会阻塞其他客户端的读，但是会阻塞写。写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写。

- 元数据锁（meta data lock，MDL）

  MDL加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。MDL锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作。为了避免DML与DDL冲突，保证读写的正确性。

  在 MySQL5.5 中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁（共享）；当对表结构进行变更操作的时候，加MDL写锁（排他）。

   ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230707215127886.png)

  查看元数据锁：

  ```mysql
  select object_type,object_schema,object_name,lock_type,lock_duration from performance_schema.metadata_locks;
  ```

- 意向锁

  为了避免 DML 在执行时，加的行锁与表锁的冲突，在 InnoDB 中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

  1. 意向共享锁（IS）：由语句 select ... lock in share mode添加，与表锁共享锁(read)兼容，与表锁排它锁(write)互斥。
  2. 意向排他锁（IX）：由 insert、update、delete、select ... for update 添加，与表锁共享锁(read)及排它锁(write)都互斥。意向锁之间不会互斥。

  可通过以下 SQL，查看意向锁及行锁的加锁情况：

  ```mysql
  select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
  ```

### 4.行级锁

- 介绍

  行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在 InnoDB 存储引擎中。

  InnoDB 的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加的锁。对于行级锁，主要分为以下三类：

  1. 行锁（Record Lock）：锁定单个行记录的锁，防止其他事务对此行进行 update 和 delete。在RC、RR 隔离级别下都支持。
  2. 间隙锁（Gap Lock）：锁定索引记忌间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。在RR隔离级别下都支持。
  3. 临键锁（Next-Key Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持。

- 行锁

  InnoDB.实现了以下两种类型的行锁：
  1.共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。
  2.排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁。
  
  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230707222631598.png)
  
  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230707222910637.png)
  
  默认情况下，InnoDB在REPEATABLE READ事务隔离级别运行，InnoDB使用next-key锁进行搜索和索引扫描，以防止幻读。
  
  1. 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
  2. InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，此时**就会升级为表锁**。
  
- 间隙锁/临键锁

  默认情况下，InnoDB 在 REPEATABLE READ事务隔离级别运行，InnoDB使用 next-key 锁进行搜索和索引扫描，以防止幻读。

  1. 索引上的等值查询（唯一索引），给不存在的记录加锁时，优化为间隙锁。
  2.  索引上的等值查询（普通索引），向右遍历时最后一个值不满足查询需求时，next-key lock 退化为间隙锁。
  3. 索引上的范围查询（唯一索引），会访问到不满足条件的第一个值为止。

  **注意：间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。**

## 六、InnoDB引擎

### 1.逻辑存储结构

- 表表空间（ibd文件），一个 mysql 实例可以对应多个表空间，用于存储记录、索引等数据。
- 段，分为数据段（Leaf node segment）、索引段（Non-leaf node segment）、回滚段（Rollback segment），InnoDB
  是索引组织表，数据段就是B+树的叶子节点，索引段即为B+树的非叶子节点。段用来管理多个Exten（区）。
- 区，表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区中一共有64个连续的页。
- 页，是InnoDB存储引擎磁盘管理的最小单元，每个页的大小默认为16KB。为了保证页的连续性，InnoDB存储引擎每次从
  磁盘申请4-5个区。
- 行，InnoDB 存储引擎数据是按行进行存放的。

### 2.架构

MySQL 5.5版本开始，默认使用InnoDB存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。下面是InnoDB架构图，左侧为内存结构，右侧为磁盘结构。

![InnoDB架构图](https://cdn.jishuqin.cn//MySQL-高级/image-20230708095029258.png)

**内存架构**

- Buffer Pool：缓冲池是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘口，加快处理速度。

- 缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型：

  1. free page：空闲page，未被使用。
  2. clean page：被使用page，数据没有被修改过。
  3. dirty page：脏页，被使用page，数据被修改过，页中数据与磁盘的数据产生了不一致。

- Change Buffer：更改缓冲区（针对于非唯一二级索引页），在执行DML语句时，如果这些数据Page没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区Change Buffer中，在未来数据被读取时，再将数据合并恢复到Buffer Poolr中，再将合并后的数据刷新到磁盘中。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230708100434924.png)

- Adaptive Hash Index：自适应hash索引，用于优化对 Buffer Pool 数据的查询。InnoDB 存储引擎会监控对表上各索引页的查询，如果观察到 hash 索引可以提升速度，则建立 hash 索引，称之为自适应 hash 索引。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230708100728834.png)

- Log Buffer：日志缓冲区，用来保存要写入到磁盘中的 log 日志数据（redo log、undo log），默认大小为16MB，日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘 I/0。

  参数：innodb_log_buffer_size：缓冲区大小	innodb_flush_log_at_trx_commit：日志刷新到磁盘时机

**磁盘结构**

- System Tablespace：系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据。（在 MySQL5.x 版本中还包含 InnoDB 数据字典、undolog等）

  参数：innodb_data_file_path

- File-Per-Table Tablespaces：每个表的文件表空间包含单个InnoDB表的数据和索引，并存储在文件系统上的单个数据文件中。
  参数：innodb_file_per_table

- General Tablespaces：通用表空间，需要通过 CREATE TABLESPACE 语法创建通用表空间，在创建表时，可以指定该表空间。

- Undo Tablespaces：撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小16M），用于存储undo log日志。

- Temporary Tablespaces：InnoDB使用会话临时表空间和全局临时表空间。存储用户创建的临时表等数据。

- Doublewrite Buffer Files：双写缓冲区，innoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件中，便于系统异常时恢复数据。

- Redo Log：重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件(redo log),前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中，用于在刷新脏页到磁盘时，发生错误时，进行数据恢复使用。

**后台线程**

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230708104200904.png)

1. Master Thread

   核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中，保持数据的一致性，还包括脏页的刷新、合并插入缓存、undo 页的回收。

2. IO Thread

   在InnoDB存储引擎中大量使用了AIO来处理IO请求，这样可以极大地提高数据库的性能，而IO Thread主要负责这些IO请求的回调。

   ![image-20230708104625112](https://cdn.jishuqin.cn//MySQL-高级/image-20230708104625112.png)

3. Purge Thread

   主要用于回收事务已经提交了的 undo log，在事务提交之后，undo log 可能不用了，就用它来回收。

4. Page Cleaner Thread

   协助 Master Thread 刷新脏页到磁盘的线程，它可以减轻 Master Thread 的工作压力，减少阻塞。

### 3.事务原理

#### 事务

事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

#### 特性

- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230708110027460.png)

#### redo log

重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性。
该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log file）,前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中，用于在刷新脏页到磁盘，发生错误时，进行数据恢复使用。

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230708111140046.png)

#### undo log

回滚日志，用于记录数据被修改前的信息，作用包含两个：提供回滚和 MVCC(多版本并发控制)。
undo log 和 redo log  记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，undo log中会记录一条对应的 insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚。
Undo log 销毁：undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些日志可能还用于MVCC。
Undo log 存储：undo log:采用段的方式进行管理和记录，存放在前面介绍的rollback segment 回滚段中，内部包含1024个undo log
segment。

### 4.MVCC

#### 基本概念

- 当前读

  读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：
  select ... lock in share mode(共享锁)，select ... for update、update、insert、delete(排他锁)都是一种当前读。

- 快照读

  简单的select(不加锁)就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。
  Read Committed：每次select,都生成一个快照读。
  Repeatable Read：开启事务后第一个selecti语句才是快照读的地方。
  Serializable：快照读会退化为当前读。

- MVCC

  全称M2ulti-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log日志、readView。
  
- readview

  ReadView（读视图）是**快照读**SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709105915084.png)

  不同的隔离级别，生成ReadView的时机不同：

  - READ COMMITTED：在事务中每一次执行快照读时生成 ReadView。
  - REPEATABLE READ：仅在事务中第一次执行快照读时生成ReadView，后续复用该 ReadView。

## 七、MySQL管理

### 1.系统数据库

MySQL 数据库安装完成后，自带了以下四个数据库，具体作用如下：

![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709152834292.png)

### 2.常用工具

- mysql 客户端工具

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709153318612.png)

- mysqladmin

  mysqladmin是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。

  通过帮助文档查看选项：

  ```shell
  mysqladmin --help
  ```

  示例：

  ```mysql
  mysqladmin -uroot -p1234 drop 'test01';
  mysqladmin -uroot -p1234 version;
  ```

- mysqlbinlog

  由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog日志管理工具。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709154610864.png)

- mysqlshow

  mysqlshow客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709155235822.png)

- mysqldump

  mysqldump 客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的sQL语句。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709160129966.png)
  
- mysqlimport/source

  mysqlimport是客户端数据导入工具，用来导入mysqldump加 -T 参数后导出的文本文件。

  ![](https://cdn.jishuqin.cn//MySQL-高级/image-20230709161523034.png)

  如果需要导入 sql 文件，可以使用 mysql 中的source指令：

  ```shell
  source /root/xxxx.sql
  ```