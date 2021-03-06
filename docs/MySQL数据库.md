## 数据库基础知识

### 为什么要使用数据库

传统的文件系统只存储非结构化的数据，我们要从文件系统中找到有用的信息，只能使用遍历匹配的方式，很不方便。而数据库存储的东西是事实，我们只需要向数据库发送请求。数据库引擎就可以返回答案给你，非常智能高效。

### 什么是SQL？

SQL全称结构化查询语言(Structured Query Language)，是一种是用于访问和处理数据库的标准的计算机语言。

### 什么是MySQL?

*MySQL是*My和SQL的组合，My是MySQL的联合创始人女儿的名字。

MySQL是一个关系型数据库管理系统，免费开源且可靠、可扩展。

### 数据库三大范式是什么

第一范式：确保每列具有原子性，不可拆分

![img](https://picb.zhimg.com/80/51e2689ac9416a91800e63101bee9db7_720w.jpg)

第二范式：在第一范式的基础上，非主属性完全依赖于主属性

![img](https://pic1.zhimg.com/80/2f4b4a887f6a61674a49d03d79e3fe17_720w.jpg)

第三范式：在第二范式的基础上，非主属性只依赖于主属性，达到消除传递依赖的作用

![img](https://pic2.zhimg.com/80/5b20707ff3d9afb51ef7bfda726c3e34_720w.jpg)

在设计数据库结构的时候，要尽量遵守三范式，它基本上解决了数据冗余过大，增删改异常的问题。但在实际中，往往为了性能上或者应对扩展的需要，经常做到2NF或者1NF。

### MySQL的binlog有几种录入格式？分别有什么区别

有三种格式，statement，row和mixed。

- statement模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
- row级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
- mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

此外，新版的MySQL中对row级别也做了一些优化，当表结构发生变化的时候，会记录语句而不是逐行记录。



## 引擎

### MySQL存储引擎MyISAM与InnoDB区别

InnoDB是 MySQL 默认的事务型存储引擎,实现了四个标准的隔离级别,主索引是聚簇索引,内部做了很多优化,支持真正的在线热备。

MYISAM不支持事务，不支持行级锁，只能全表加锁，读取时会对所有表加共享锁，写入时会对表加排他锁。

具体：（第二阶段回答）

是 MySQL 默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

实现了四个标准的隔离级别，默认级别是可重复读（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+ Next-Key Locking 防止幻影读。

在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。

内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。

支持真正的在线热备份。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。



### InnoDB引擎的4大特性

- 插入缓冲（insert buffer)
- 二次写(double write)
- 自适应哈希索引(ahi)
- 预读(read ahead)

## 索引

### 什么是索引？为什么要使用索引？

索引即关键字与数据的映射关系，包含关键字和对应的记录在磁盘中的地址

关键字是从数据当中提取的用于标识、检索数据的特定内容。

索引可以让服务器快速定位到表的指定位置。

## 索引检索为什么快？

- 关键字相对于数据本身，数据量小
- 关键字是有序的，二分查找可快速确定位置

### 字符串索引和数字类型索引的区别

使用数字类型索引比字符串索引快

因为字符串在传输过程中要编码和解码。而这一过程需要消耗CPU资源

如果非要用字符串索引可以采用以下解决方法。

1.对字符串的前n个字符建立前缀索引

补充：MySQL 前缀索引能有效减小索引文件的大小，提高索引的速度。但是前缀索引也有它的坏处：MySQL 不能在 ORDER BY 或 GROUP BY 中使用前缀索引，也不能把它们用作覆盖索引(Covering Index)。

2.增加一列，对字符串转换为整型的hash值address_key=hashToInt(address)，对address_key建立索引，查询时同时限定hash值也限定地址。可以用如下查询where address_key = hashToInt(‘beijing,china’) and address = ‘beijing,china’;

效率的话，100万的数据量，字符串索引查询600ms，数字查询20ms。



### union和unionall的区别是什么？

union就是将两个SELECT语句查询的结果集合并(两个SELECT可以是同一个表，也可以是不同表)，如果需要排序，在第二个SELECT语句后加ORDER BY语句，会对所有结果进行排序。

union默认是会去除重复的数据的，会对结果集做去重处理，union all不会做去重处理。

所以union效率慢一些，如果能确定结果不会重复或者需要不去重的结果，那么应该使用union all，效率会高一些。



### 适合索引的场景

1. 主键作为唯一性标识字段，需要频繁地访问和连接。建议给每张表指定主键，因为InnoDB会自动为主键建立聚集索引，即使没得主键，也会自动生成一个隐藏主键

2. 频繁作为查询条件的字段，例如where

3. 查询中常作为查询排序条件的字段，例如where加上orderby

4. 查询中的统计和分组字段，例如where加上groupby或者union。

补充：explain解释语句后会出现可选的索引和真正用来检索的索引。数据库会在可选索引中选择一个最优的索引。

（在order by中）如果不对字段建立索引，查询时会将磁盘中的所有数据读入内存，然后使用外部排序，这个过程很影响性能。

如果我们对字段建立索引，由于索引本身是有序的，所以我们只需要取出索引表某个范围内的索引对应的数据就可以了，大大减低了IO的次数，提高了索引速度。

### 不适合用索引的场景

1. 表的记录不多

2. 数据重复且分布平均的字段

3. 频繁更新的字段、表

4. 很少作为查询条件的字段

补充：如果字段添加了索引，在更新时不仅要更新数据本身，还要维护其索引，如果频繁更新会带来很多额外开销。再者，如果一个表频繁进行增删改操作，也不适合索引。一般数据量达到300万-500万时考虑建立索引。



### 索引有哪几种类型？

**主键索引:** 数据列不允许重复，不允许为NULL，一个表只能有一个主键。

**唯一索引:** 数据列不允许重复，允许为NULL值，一个表允许多个列创建唯一索引。

- 可以通过 `ALTER TABLE table_name ADD UNIQUE (column);` 创建唯一索引
- 可以通过 `ALTER TABLE table_name ADD UNIQUE (column1,column2);` 创建唯一组合索引

**普通索引:** 基本的索引类型，没有唯一性的限制，允许为NULL值。

- 可以通过`ALTER TABLE table_name ADD INDEX index_name (column);`创建普通索引
- 可以通过`ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3);`创建组合索引

**全文索引：** 是目前搜索引擎使用的一种关键技术。

- 可以通过`ALTER TABLE table_name ADD FULLTEXT (column);`创建全文索引



### 索引的基本原理

索引用来快速地寻找那些具有特定值的记录。如果没有索引，一般来说执行查询时遍历整张表。

索引的原理很简单，就是把无序的数据变成有序的查询

1. 把创建了索引的列的内容进行排序
2. 对排序结果生成倒排表
3. 在倒排表内容上拼上数据地址链
4. 在查询的时候，先拿到倒排表内容，再取出数据地址链，从而拿到具体数据

### 索引算法有哪些？

索引算法有 BTree算法和Hash算法

**BTree算法**

BTree是最常用的mysql数据库索引算法，也是mysql默认的算法。因为它不仅可以被用在=,>,>=,<,<=和between这些比较操作符上，而且还可以用于like操作符，只要它的查询条件是一个不以通配符开头的常量

**Hash算法**

Hash Hash索引只能用于对等比较。由于是一次定位数据，不像BTree索引需要从根节点到枝节点，最后才能访问到页节点这样多次IO访问，所以检索效率远高于BTree索引。

### 索引设计的原则？

1. 适合索引的列是出现在where子句中的列，或者连接子句中指定的列
2. 基数较小的类，索引效果较差，没有必要在此列建立索引
3. 使用短索引，如果对长字符串列进行索引，应该指定一个前缀长度，这样能够节省大量索引空间
4. 不要过度索引。索引需要额外的磁盘空间，并降低写操作的性能。在修改表内容的时候，索引会进行更新甚至重构，索引列越多，这个时间就会越长。所以只保持需要的索引有利于查询即可。

### 创建索引的原则（重中之重）

索引虽好，但也不是无限制的使用，最好符合一下几个原则

1） 最左前缀匹配原则，组合索引非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2）较频繁作为查询条件的字段才去创建索引

3）更新频繁字段不适合创建索引

4）若是不能有效区分数据的列不适合做索引列(如性别，男女未知，最多也就三种，区分度实在太低)

5）尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

6）定义有外键的数据列一定要建立索引。

7）对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

### 创建索引的三种方式，删除索引

第一种方式：在执行CREATE TABLE时创建索引

```sql
CREATE TABLE user_index2 (
	id INT auto_increment PRIMARY KEY,
	first_name VARCHAR (16),
	last_name VARCHAR (16),
	id_card VARCHAR (18),
	information text,
	KEY name (first_name, last_name),
	FULLTEXT KEY (information),
	UNIQUE KEY (id_card)
);
12345678910
```

第二种方式：使用ALTER TABLE命令去增加索引

```sql
ALTER TABLE table_name ADD INDEX index_name (column_list);
1
```

ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。

其中table_name是要增加索引的表名，column_list指出对哪些列进行索引，多列时各列之间用逗号分隔。

索引名index_name可自己命名，缺省时，MySQL将根据第一个索引列赋一个名称。另外，ALTER TABLE允许在单个语句中更改多个表，因此可以在同时创建多个索引。

第三种方式：使用CREATE INDEX命令创建

```sql
CREATE INDEX index_name ON table_name (column_list);
1
```

CREATE INDEX可对表增加普通索引或UNIQUE索引。（但是，不能创建PRIMARY KEY索引）

删除索引

根据索引名删除普通索引、唯一索引、全文索引：`alter table 表名 drop KEY 索引名`

```sql
alter table user_index drop KEY name;
alter table user_index drop KEY id_card;
alter table user_index drop KEY information;
123
```

删除主键索引：`alter table 表名 drop primary key`（因为主键只有一个）。这里值得注意的是，如果主键自增长，那么不能直接执行此操作（自增长依赖于主键索引）：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8yLzE5LzE2OTA0NTk2YjIxZTIwOWM?x-oss-process=image/format,png)

需要取消自增长再行删除：

```sql
alter table user_index
-- 重新定义字段
MODIFY id int,
drop PRIMARY KEY
1234
```

但通常不会删除主键，因为设计主键一定与业务逻辑无关。

### 创建索引时需要注意什么？

- 非空字段：应该指定列为NOT NULL，除非你想存储NULL。在mysql中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值；
- 取值离散大的字段：（变量各个取值之间的差异程度）的列放到联合索引的前面，可以通过count()函数查看字段的差异值，返回值越大说明字段的唯一值越多字段的离散程度高；
- 索引字段越小越好：数据库的数据存储以页为单位一页存储的数据越多一次IO操作获取的数据越大效率越高。

### 使用索引查询一定能提高查询的性能吗？为什么

不一定！一般来说，通过索引查询数据比全表扫描要快，但是如果我们建立过多不必要的索引或者错误地使用了索引，就不一定能提高查询的性能！

补充：索引需要空间来储存且需要定期维护。索引在基于范围的检索中结果集不准确且在非唯一性索引时较慢。

### 百万级别或以上的数据如何删除

我们对数据的增删改会产生对索引文件的操作，这些操作需要消耗额外的IO。由于在删除数据库百万级别的数据时，删除数据的速度和创建的索引数量成正比。所以我们的方案为先删除索引，然后删除无用数据，最后重新创建索引回归工作状态。

这样做的好处相比于直接删除来说：减少了删除时间，最大可能性避免删除中断造成回滚。

### 使用前缀索引需要注意什么？

注意点一：前缀的标识度要高。比如密码就适合建立前缀索引，因为密码几乎各不相同。

注意点二：前缀的截取长度适宜

补充：

语法：`index(field(10))`，使用字段值的前10个字符建立索引，默认是使用字段的全部内容建立索引。

我们可以利用通过调整`prefixLen`的值查看不同前缀长度的一个平均匹配度，接近1时就可以了。

### 什么是最左前缀原则？什么是最左匹配原则

最左前缀原则，就是最左优先，在创建多列索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。

最左前缀匹配原则，mysql会一直向右匹配直到遇到范围查询就停止匹配。

补充：

=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

### 聚簇索引和非聚簇索引的区别？使用聚簇索引的优缺点？

最大的区别为叶子节点是否存放一整行数据

1.对于聚簇索引，表数据和主键是一起存储的，主键索引的叶子节点存储的是主键值和对应的一整行数据，二级索引的叶子节点存储的是对应字段和主键值。

2.对于非聚簇索引，表数据和索引是分成存储的，主键索引和二级索引的叶子节点存放的是索引和指向索引对应记录数据的指针。

聚簇索引的优点：

1.当你需要取出一定范围内的数据时，用聚簇索引也比用非聚簇索引好。

2.当通过聚簇索引查找目标数据时理论上比非聚簇索引要快，因为非聚簇索引定位到对应主键时还要多一次目标记录寻址,即多一次I/O。

3.使用覆盖索引扫描的查询可以直接使用叶子节点中的主键值。

聚簇索引的缺点：

1.二级索引访问需要两次索引查找，第一次找到主键值，第二次根据主键值找到行数据。

2.采用聚簇索引插入新值比采用非聚簇索引插入新值的速度要慢很多

补充：

因为插入要保证主键不能重复，而采用的方式在不同的索引下面会有很大的性能差距，聚簇索引和非聚簇索引都会去判断所有的叶子节点，但是聚簇索引的叶子节点除了带有主键还有记录值，记录的大小往往比主键要大的多。这样就会导致聚簇索引在判定新记录携带的主键是否重复时进行昂贵的I/O代价。

### B树和B+树的区别

- 在B树中，你可以将键和值存放在内部节点和叶子节点；但在B+树中，内部节点都是键，没有值，叶子节点同时存放键和值。

- B+树的叶子节点有一条链相连，而B树的叶子节点各自独立。

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC85LzIxLzE2NWZiNjgyZTc1OWNmMTI?x-oss-process=image/format,png)

### 使用B树的好处

B树可以在内部节点同时存储键和值，因此，把频繁访问的数据放在靠近根节点的地方将会大大提高热点数据的查询效率。这种特性使得B树在特定数据重复多次查询的场景中更加高效。

### 与红黑树相比使用B+树的好处

1. 更少的查找次数
2. 更快地读取速度
3. 存储更多的索引节点

补充：磁盘预读特性：数据库请求数据的时候，会将读请求交给文件系统，放入请求队列中；相关进程从请求队列中将读请求取出，根据需求到相关数据区(内存、磁盘)读取数据；取出的数据，放入响应队列中，最后数据库就会从响应队列中将数据取走，完成一次数据读操作过程。

### 与B树相比使用B+树的好处

在节点存储内容上：

由于B+树的内部节点只存放键不存放值，因此一次读取可以在内存页中获取更多的键，有利于更快地缩小查找范围。

在叶子节点上：

B+树的叶节点由一条链相连，因此，当需要进行一次全数据遍历的时候，B+树只需要使用O(logN)时间找到最小的一个节点，然后通过链进行O(N)的顺序遍历即可。

而B树则需要对树的每一层进行遍历，这会需要更多的内存置换次数，因此也就需要花费更多的时间



### MySQL 默认的存储引擎选择 B+ 树而不是哈希或者 B 树的原因？

- 哈希虽然能够提供 O(1) 的单数据行操作性能，但是对于范围查询和排序却无法很好地支持，最终导致全表扫描；
- B 树能够在非叶节点中存储数据，但是这也导致在查询连续数据时可能会带来更多的随机 I/O，而 B+ 树的所有叶节点可以通过指针相互连接，能够减少顺序遍历时产生的额外随机 I/O；

总和考虑，B+ 树可能无法对所有 OLTP 场景下的查询都有着较好的性能，但是它能够解决大多数的问题。软件工程中没有银弹，所以我们在选择数据库时也应该非常清楚地知道不同数据库适合的场景。

### 聚簇索引、辅助索引和联合索引分别是什么？

数据要想有规律地组织起来就必须建立索引。

情景一：

Mysql无论如何都会建立一个存储完整行数据的索引，就叫做聚簇索引。

聚簇索引会根据主键或者唯一键建立一个B+树，树的层数决定了你访问磁盘的次数。

情景二：

```mysql
create index idx_name on student(name);
```

为了避免查询普通键时触发全表扫描，建立一个存储部分行数据的索引，就叫做二级索引，也叫做辅助索引。

新建的这棵树，叶子节点存储主键ID和普通键，匹配成功后拿着主键ID去聚簇索引中找就可以了，也叫回表操作

情景三：

```sql
create index idx_name_age on student(name,age);
```

将姓名和年龄同时建索引，此时内节点中遵循规律。先按照name排序，如果name相同，则按照age排序。

### 非聚簇索引一定会回表查询吗？

不一定，这涉及到查询语句所要求的字段是否全部命中了索引，如果全部命中了索引，那么就不必再进行回表查询。

举个简单的例子，假设我们在员工表的年龄上建立了索引，那么当进行`select age from employee where age < 20`的查询时，在索引的叶子节点上，已经包含了age信息，不会再次进行回表查询。

### 联合索引是什么？为什么需要注意联合索引中的顺序？

MySQL可以使用多个字段同时建立一个索引，叫做联合索引。在联合索引中，如果想要命中索引，需要按照建立索引时的字段顺序挨个使用，否则无法命中索引。

具体原因为:

MySQL使用索引时需要索引有序，假设现在建立了"name，age，school"的联合索引，那么索引的排序为: 先按照name排序，如果name相同，则按照age排序，如果age的值也相等，则按照school进行排序。

当进行查询时，此时索引仅仅按照name严格有序，因此必须首先使用name字段进行等值查询，之后对于匹配到的列而言，其按照age字段严格有序，此时可以使用age字段用做索引查找，以此类推。因此在建立联合索引的时候应该注意索引列的顺序，一般情况下，将查询需求频繁或者字段选择性高的列放在前面。此外可以根据特例的查询或者表结构进行单独的调整。

## 事务

### 什么是数据库事务？

事务是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。

事务最经典的例子就是银行转账

### 事物的四大特性(ACID)介绍一下?

关系性数据库需要遵循ACID规则，具体内容如下：

![事务的特性](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC81LzIwLzE2MzdiMDhiOTg2MTk0NTU?x-oss-process=image/format,png)

1. **原子性：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. **一致性：** 执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；
3. **隔离性：** 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
4. **持久性：** 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

### 什么是脏读？幻读？不可重复读？

- 脏读(Drity Read)：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。
- 不可重复读(Non-repeatable read):在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。
- 幻读(Phantom Read):在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。

### 什么是事务的隔离级别？MySQL的默认隔离级别是什么？

为了达到事务的四大特性，数据库定义了4种不同的事务隔离级别，由低到高依次为Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。

| 隔离级别         | 脏读 | 不可重复读 | 幻影读 |
| ---------------- | ---- | ---------- | ------ |
| READ-UNCOMMITTED | √    | √          | √      |
| READ-COMMITTED   | ×    | √          | √      |
| REPEATABLE-READ  | ×    | ×          | √      |
| SERIALIZABLE     | ×    | ×          | ×      |

**SQL 标准定义了四个隔离级别：**

- **READ-UNCOMMITTED(读取未提交)：** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。
- **READ-COMMITTED(读取已提交)：** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。
- **REPEATABLE-READ(可重复读)：** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。
- **SERIALIZABLE(可串行化)：** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。

这里需要注意的是：Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别

事务隔离机制的实现基于锁机制和并发调度。其中并发调度使用的是MVVC（多版本并发控制），通过保存修改的旧版本信息来支持并发一致性读和回滚等特性。

因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是**READ-COMMITTED(读取提交内容):**，但是你要知道的是InnoDB 存储引擎默认使用 **REPEATABLE-READ（可重读）**并不会有任何性能损失。

InnoDB 存储引擎在 **分布式事务** 的情况下一般会用到**SERIALIZABLE(可串行化)**隔离级别。

## 锁

### 对MySQL的锁了解吗

当数据库有并发事务的时候，可能会产生数据的不一致，这时候需要一些机制来保证访问的次序，锁机制就是这样的一个机制。



### MySQL中InnoDB引擎的行锁是怎么实现的？

答：InnoDB是基于索引来完成行锁

例: select * from tab_with_index where id = 1 for update;

for update 可以根据条件来完成行锁锁定，并且 id 是有索引键的列，如果 id 不是索引键那么InnoDB将完成表锁，并发将无从谈起

### InnoDB存储引擎的锁的算法有三种

- Record lock：单个行记录上的锁
- Gap lock：间隙锁，锁定一个范围，不包括记录本身
- Next-key lock：record+gap 锁定一个范围，包含记录本身

**相关知识点：**

1. innodb对于行的查询使用next-key lock
2. Next-locking keying为了解决Phantom Problem幻读问题
3. 当查询的索引含有唯一属性时，将next-key lock降级为record key
4. Gap锁设计的目的是为了阻止多个事务将记录插入到同一范围内，而这会导致幻读问题的产生
5. 有两种方式显式关闭gap锁：（除了外键约束和唯一性检查外，其余情况仅使用record lock） A. 将事务隔离级别设置为RC B. 将参数innodb_locks_unsafe_for_binlog设置为1

### 什么是死锁？怎么解决？

死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方的资源，从而导致恶性循环的现象。

常见的解决死锁的方法

1、如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。

2、在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率；

3、对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率；

如果业务处理不好可以用分布式事务锁或者使用乐观锁

### 数据库的乐观锁和悲观锁是什么？怎么实现的？

数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。乐观并发控制（乐观锁）和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。

**乐观锁**：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过version的方式来进行锁定。实现方式：一般会使用版本号机制或CAS算法实现。

**悲观锁**：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。在查询数据的时候就把事务锁起来，直到提交事务。实现方式：使用数据库中的锁机制

**两种锁的使用场景**

乐观锁适用于多读场景，悲观锁适用于多写场景

### 六种关联查询

- 交叉连接（CROSS JOIN）
- 内连接（INNER JOIN）
- 外连接（LEFT JOIN/RIGHT JOIN）
- 联合查询（UNION与UNION ALL）
- 全连接（FULL JOIN）
- 交叉连接（CROSS JOIN）



### 什么是子查询？

1. 条件：一条SQL语句的查询结果做为另一条查询语句的条件或查询结果
2. 嵌套：多条SQL语句嵌套使用，内部的SQL查询语句称为子查询。

### 子查询的三种情况

1. 子查询是单行单列的情况：结果集是一个值，父查询使用：=、 <、 > 等运算符

```sql
-- 查询工资最高的员工是谁？ 
select  * from employee where salary=(select max(salary) from employee);   
12
```

1. 子查询是多行单列的情况：结果集类似于一个数组，父查询使用：in 运算符

```sql
-- 查询工资最高的员工是谁？ 
select  * from employee where salary=(select max(salary) from employee);    
12
```

1. 子查询是多行多列的情况：结果集类似于一张虚拟表，不能用于where条件，用于select子句中做为子表

```sql
-- 1) 查询出2011年以后入职的员工信息
-- 2) 查询所有的部门信息，与上面的虚拟表中的信息比对，找出所有部门ID相等的员工。
select * from dept d,  (select * from employee where join_date > '2011-1-1') e where e.dept_id =  d.id;    

-- 使用表连接：
select d.*, e.* from  dept d inner join employee e on d.id = e.dept_id where e.join_date >  '2011-1-1'  
123456
```

### mysql中 in 和 exists 区别

mysql中的in语句是把外表和内表作hash 连接，而exists语句是对外表作loop循环，每次loop循环再对内表进行查询。一直大家都认为exists比in语句的效率要高，这种说法其实是不准确的。这个是要区分环境的。

1. 如果查询的两个表大小相当，那么用in和exists差别不大。
2. 如果两个表中一个较小，一个是大表，则子查询表大的用exists，子查询表小的用in。
3. not in 和not exists：如果查询语句使用了not in，那么内外表都进行全表扫描，没有用到索引；而not extsts的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快。

### varchar与char的区别

**char的特点**

- char表示定长字符串，长度是固定的；
- 如果插入数据的长度小于char的固定长度时，则用空格填充；
- 因为长度固定，所以存取速度要比varchar快很多，甚至能快50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；
- 对于char来说，最多能存放的字符个数为255，和编码无关

**varchar的特点**

- varchar表示可变长字符串，长度是可变的；
- 插入的数据是多长，就按照多长来存储；
- varchar在存取方面与char相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- 对于varchar来说，最多能存放的字符个数为65532

总之，结合性能角度（char更快）和节省磁盘空间角度（varchar更小），具体情况还需具体来设计数据库才是妥当的做法。

### drop、delete与truncate的区别

三者都表示删除，但是三者有一些差别：

delete属于DML类型，用于删除表的全部或者部分数据行，表结构还在，可回滚、删除速度慢

truncate属于DDL类型，用于删除表中所有数据，表结构还在，不可回滚、删除速度快

drop属于DDL类型，用于删除表，表结构不在，不可回滚，删除速度最快。

因此，在不再需要一张表的时候，用drop；在想删除部分数据行时候，用delete；在保留表而删除所有数据的时候用truncate。

### 日常调优的流程是怎么样的？

涉及SQL的业务区本地环境中跑一遍，用explain去看一下执行计划，看看分析的结果是否符合自己的预期，用没用到相关的索引，然后去线上环境泡一下看下执行时间（这里指的是查询语句）

### 有哪些方向可以优化？

排除缓存干扰

explain看下执行计划：

1. 利用覆盖索引减少回表操作
2. 建立联合索引进行多分段查询
3. 利用好最左匹配原则和索引下推
4. 合理选择普通索引和唯一索引
5. 对于长字符串索引利用好前缀索引

### MySQL45讲中的问题？

一条语句内部的执行过程是怎么样的？

一条sql更新语句的执行是怎么样的？

你了解事务隔离吗？

说说索引？

谈谈全局锁、表锁、行锁

事务到底是隔离的还是不隔离的？

如何正确选择普通索引还是唯一索引？

MySQL为什么会选错索引？

怎么给字符串字段加索引？

为什么我的MySQL会抖一下？

为什么表数据删掉一半，表文件的大小不变？

count(星)很慢该怎么办？

