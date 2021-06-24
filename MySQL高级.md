## MySQL高级

### 一、`linux`上`mysql`的基本命令

#### 1、安装命令

* 查看版本：`mysqladmin --version`

  ```cmd
  root@lin-NH5x-7xRCx-RDx:/home/lin/桌面# mysqladmin --version
  mysqladmin  Ver 8.42 Distrib 5.7.33, for Linux on x86_64
  ```

* 设置用户和密码：`mysqladmin -u root password`

  ![image-20210321164656659](./pics/MySQL基础.md)

#### 2、`mysql`服务命令

查看状态：`service mysql status`

启动服务：`service mysql start`

停止服务：`service mysql stop`

重启服务：`service mysql restart`

#### 3、设置大小写不敏感（linux下mysql默认是大小写敏感的）

查看大小写是否敏感：`show variables like '%lower_case_table_names%'`

设置大小写不敏感：在配置文件 [mysqld] 中加入 lower_case_table_names = 1 ,然后重启服务器

#### 4、进入MySQL数据库

```cmd
$ mysql -u root -p
$ mysql -h localhost -u root -p
```

> -u 表示选择登陆的用户名
> -p 表示登陆的用户密码
> -h 登录主机名

#### 5、常用命令

| SQL语句                                           | 描述                             | 备注                                      |
| ------------------------------------------------- | -------------------------------- | ----------------------------------------- |
| show  databases                                   | 列出所有数据库                   |                                           |
| create database 库名use                           | 创建一个数据库                   |                                           |
| create database 库名 character set utf8           | 创建数据库,顺便执行字符集为utf-8 |                                           |
| show create database 库名                         | 查看数据库的字符集               |                                           |
| show variables like ‘%char%’                      | 查询所有跟字符集相关的信息       |                                           |
| set [字符集属性]=utf8                             | 设置相应的属性为 utf8            | 只是临时修改,当前有效。服务重启后，失效。 |
| alter database 库名 character set 'utf8'          | 修改数据库的字符集               |                                           |
| alter table 表 名 convert to character set 'utf8' | 修改表的字符集                   |                                           |



### 二、`Mysql`的用户权限和权限管理

#### 1、`Mysql`的用户管理

##### 1.1、相关命令

| 命令                                                         | 描述                                      | 备注                                                         |
| ------------------------------------------------------------ | ----------------------------------------- | ------------------------------------------------------------ |
| `create user zhang3 identified by '123123';`                 | 创建名称为 zhang3 的用户,密码设为 123123; |                                                              |
| `select host,user,password,select_priv,insert_priv,drop_priv from mysql.user;` | 查看用户和权限的相关信息                  |                                                              |
| `set password =password('123456')`                           | 修改当前用户的密码                        |                                                              |
| `update mysql.user set password=password('123456') where user='li4';` | 修改其他用户的密码                        | 所有通过 user 表的修改,必须用 flush privileges; 命 令 才 能 生效 |
| `update mysql.user set user='li4' where user='wang5';`       | 修改用户名                                | 所有通过 user 表的修改,必须用 flush privileges; 命令才能生效 |
| `drop user li4`                                              | 删除用户                                  |                                                              |

#### 2、`Mysql`的权限管理

##### 2.1、授予权限（grant）

| 命令                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `grant 权限1,权限2,...权限n on 数据库名.表名 to 用户名@用户地址 identified by '连接口令'` | 该权限如果发现没有该用户,则会直接新建一个用户                |
| `grant all privileges on *.* to joe@'%' identified by '123'` | 授予通过网络方式登录的的 joe 用户 ,对所有库所有表的全部权限,密码设为 123. |

##### 2.2、收回权限（revoke）

| 命令                                                         | 描述                                   |
| ------------------------------------------------------------ | -------------------------------------- |
| `show grants`                                                | 查看当前用户权限                       |
| `revoke [权限1,权限2,...权限n] on 数据库名.表名 from 用户名@用户地址` | 回收权限命令                           |
| `REVOKE ALL PRIVILEGES ON mysql.* FROM joe@localhost;`       | 收回全库全表的所有权限                 |
| `REVOKE select,insert,update,delete ON mysql.* FROM joe@localhost;` | 收回 mysql 库下的所有表的插删改查 权限 |

注：权限回收后，必须用户重新登录后才能生效

### 三、`Mysql`逻辑架构简介

#### 1、整体架构图

![image-20210321173014626](/home/lin/桌面/pics/image-20210321173014626.png)

和其它数据库相比,`MySQL `有点与众不同,它的架构可以在多种不同场景中应用并发挥良好作用。**主要体现在存储引擎的架构上,插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。**

##### 1.1、连接层

最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务器工具实现的类似于`tcp/ip`的通信。主要完成一些类似于连接处理、授权认证及相关的安全方案。在该层上引入拉线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于`SSL`的安全链接。服务器也会为安全接入每个客户端验证它所具有的操作权限

##### 1.2、服务层

| 服务层                         |                                                              |
| ------------------------------ | ------------------------------------------------------------ |
| Management Service & Utilities | 系统管理和控制工具                                           |
| `SQL interface`                | `SQL`接口。接受用户的`SQL`命令，并且返回用户需要查询的结果。比如select from就是调用`SQL interface` |
| Parser                         | 解析器                                                       |
| Optimizer                      | 查询优化器。`SQL`语句在查询之前会使用查询优化器对查询进行优化，比如有where条件时，优化器来决定是先投影还是先过滤 |
| Cache和Buffer                  | 查询缓存。如果查询缓冲有命中查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等 |

 ##### 1.3、引擎层

存储引擎层，存储引擎真正的负责了`MySQL`中数据的存储和提取，服务器通过`API`与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。

##### 1.4、存储层

数据存储层，主要将数据存储运行在裸设备的文件系统之上，并完成与存储引擎的交互

#### 2、show profile

* 利用show profile可以查看sql的执行周期！

##### 2.1、开启profile

`show variables like '%profiling'`

如果没有开启,可以执行 set profiling=1 开启!

##### 2.2、使用profile

执行`show profiles`命令，可以查看最近几次的查询

##### 2.3、大致的查询流程

* mysql客户端通过协议与mysql服务器建立连接，发送查询语句，先检查缓存，如果命中，直接返回结果，否则进行语句解析。
* 语法解析和预处理：首先mysql通过关键字将sql语句进行解析，并生成一颗对应的“解析树”，mysql解析器将使用mysql语法规则验证和解析查询;预处理器则根据一些mysql规则进一步检查解析树是否合法
* 查询优化器当解析优化被认为是合法的了，并且由优化器将其转化成执行计划，一条查询可以有很多执行方式，最后都返回相同的结果，优化器的作用就是找到这其中最好的执行计划。
* mysql默认使用BTREE索引，并且一个大致方向是：无论怎么折腾sql，至少在目前来说，mysql最多只用到表中的一个索引

##### 2.4、SQL的执行顺序

手写顺序

![image-20210404142543097](/home/lin/桌面/pics/image-20210404142543097.png)

真正执行的顺序

> 随着MySQL版本的更新换代，其优化器也在不断的升级，优化器会分析不用执行顺序产生的性能消耗不同而动态调整执行顺序，常见的查询顺序如下

![image-20210404142829819](/home/lin/桌面/pics/image-20210404142829819.png)

![image-20210404142850775](/home/lin/.config/Typora/typora-user-images/image-20210404142850775.png)

##### 2.5、MyISAM和InnoDB

| 对比项         | MyISAM                                             | InnoDB                                                       |
| -------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| 外键           | 不支持                                             | 支持                                                         |
| 事务           | 不支持                                             | 支持                                                         |
| 行表锁         | 表锁，即使操作一条记录也会锁住整个表，不适合高并发 | 行锁，操作时只锁某一行，不对其它行有影响，适合高并发操作     |
| 缓存           | 只缓存索引，不缓存真实数据                         | 不仅缓存索引还要缓存真实数据，对内存要求高，而且内存大小对性能有决定性的影响 |
| 关注点         | 读性能                                             | 并发写、事务、资源                                           |
| 默认安装       | Y                                                  | Y                                                            |
| 默认使用       | N                                                  | Y                                                            |
| 自带系统表使用 | Y                                                  | N                                                            |

> 查看所有数据库引擎：`show engines`
>
> 查看默认的数据库引擎：`show variables like '%storage_engine'`

### 四、SQL预热

#### 1、常见的Join查询图

![image-20210404150834351](/home/lin/桌面/pics/image-20210404150834351.png)

#### 2、Join示例

##### 2.1、建表语句

```sql
CREATE TABLE `t_dept` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`deptName` VARCHAR(30) DEFAULT NULL,
	`address` VARCHAR(40) DEFAULT NULL,
	PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
CREATE TABLE `t_emp` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(20) DEFAULT NULL,
	`age` INT(3) DEFAULT NULL,
	`deptId` INT(11) DEFAULT NULL,
	empno int not null,
	PRIMARY KEY (`id`),
	KEY `idx_dept_id` (`deptId`)
	#CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) 	REFERENCES `t_dept` (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


INSERT INTO t_dept(deptName,address) VALUES('华山','华山');
INSERT INTO t_dept(deptName,address) VALUES('丐帮','洛阳');
INSERT INTO t_dept(deptName,address) VALUES('峨眉','峨眉山');
INSERT INTO t_dept(deptName,address) VALUES('武当','武当山');
INSERT INTO t_dept(deptName,address) VALUES('明教','光明顶');
INSERT INTO t_dept(deptName,address) VALUES('少林','少林寺');
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('风清扬',90,1,100001);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('岳不群',50,1,100002);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('令狐冲',24,1,100003);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('洪七公',70,2,100004);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('乔峰',35,2,100005);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('灭绝师太',70,3,100006);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('周芷若',20,3,100007);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('张三丰',100,4,100008);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('张无忌',25,5,100009);
INSERT INTO t_emp(NAME,age,deptId,empno) VALUES('韦小宝',18,null,100010);
```

##### 2.2、案例

> 所有有门派人员的信息(要求显示门派名称)

```sql
SELECT e.`name`,d.`deptName` FROM t_emp e INNER JOIN t_dept d ON e.`deptId`=d.`id`;
```

> 列出所有人员及其门派信息

```sql
SELECT e.`name`,d.`deptName` FROM t_emp e LEFT JOIN t_dept d ON e.`deptId`=d.`id`;
```

> 列出所有门派

```sql
SELECT * FROM t_dept;
```

> 所有无门派人士

```sql
SELECT * FROM t_emp WHERE deptId IS NULL;
```

> 所有无人门派

```sql
SELECT d.* FROM
t_dept d LEFT JOIN t_emp e ON d.`id`=e.`deptId` WHERE e.`deptId` IS NULL;
```

> 所有人员和门派的对应关系

```sql
SELECT * FROM t_emp e LEFT JOIN t_dept d ON e.`deptId`=d.`id`
UNION
SELECT * FROM t_emp e RIGHT JOIN t_dept d ON e.`deptId`=d.`id`;
```

> 所有没有入门派的人员和没人入的门派

```sql
SELECT * FROM t_emp e
LEFT JOIN t_dept d ON e.`deptId`=d.`id` WHERE e.deptId IS NULL
UNION
SELECT * FROM
t_dept d LEFT JOIN t_emp e ON d.`id`=e.`deptId` WHERE e.`deptId` IS NULL;
```

> 添加 CEO 字段

```sql
ALTER TABLE `t_dept`
add CEO
INT(11);
update t_dept set CEO=2 where id=1;
update t_dept set CEO=4 where id=2;
update t_dept set CEO=6 where id=3;
update t_dept set CEO=8 where id=4;
update t_dept set CEO=9 where id=5;
```

> 求各个门派对应的掌门人名称

```sql
SELECT d.deptName,e.name FROM t_dept d LEFT JOIN t_emp e ON d.ceo=e.id
```

> 求所有当上掌门人的平均年龄

```sql
SELECT AVG(e.age) FROM t_dept d LEFT JOIN t_emp e ON d.ceo=e.id
```

> 求所有人物对应的掌门名称

```sql
SELECT ed.name '人物',c.name '掌门' FROM
(SELECT e.name,d.ceo from t_emp e LEFT JOIN t_dept d on e.deptid=d.id) ed
LEFT JOIN t_emp c on ed.ceo= c.id;
```

### 五、索引

#### 1、索引的概念？

##### 1.1、是什么？

索引(Index)是帮助 MySQL 高效获取数据的数据结构。可以得到索引的本质:**索引是数据结构**。可以简单理解为排好序的快速查找数据结构。

![image-20210407095749422](/home/lin/桌面/pics/image-20210407095749422.png)

一般来说索引本身也很大,不可能全部存储在内存中,因此索引往往以索引文件的形式存储的磁盘上。

##### 1.2、优缺点

优势：

* 提高数据检索的效率，降低数据库的io成本
* 通过索引列对数据进行排序，降低数据排序成本，降低拉CPU的消耗

劣势：

* 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE、DELETE。因为表更新时，MySQL不仅要保存数据，还要保存以下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息
* 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引也是要占用空间的。

#### 2、`MySQL`的索引

##### 2.1、`BTree`索引

`MySQL`使用的是`BTree`索引：

![image-20210407100638870](/home/lin/桌面/pics/image-20210407100638870.png)

> 初始化介绍
>
> 一颗b树，浅蓝色的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色）和指针（黄色）

如磁盘块 1 包含数据项 17 和 35,包含指针 `P1、P2、P3`

注意：

* **真实的数据存在于叶子节点**即 3、5、9、10、13、15、28、29、36、60、75、79、90、99。
* **非叶子节点不存真实的数据**，只存指定搜索方向的数据项，如17、35并不真实存在于数据表中

> 查找过程
>
> 如果要查找数据项 29,那么首先会把磁盘块 1 由磁盘加载到内存,此时发生一次 IO,在内存中用二分查找确定 29在 17 和 35 之间,锁定磁盘块 1 的 P2 指针,内存时间因为非常短(相比磁盘的 IO)可以忽略不计,通过磁盘块 1的 P2 指针的磁盘地址把磁盘块 3 由磁盘加载到内存,发生第二次 IO,29 在 26 和 30 之间,锁定磁盘块 3 的 P2 指针,通过指针加载磁盘块 8 到内存,发生第三次 IO,同时内存中做二分查找找到 29,结束查询,总计**三次 IO**。
>
>  
>
> 例如：3 层的 b+树可以表示上百万的数据,如果上百万的数据查找只需要三次 IO,性能提高将是巨大的,如果没有索引,每个数据项都要发生一次 IO,那么总共需要百万次的 IO,显然成本非常非常高。

##### 2.2、B+Tree索引

![image-20210407102919398](/home/lin/桌面/pics/image-20210407102919398.png)

**B+Tree 与 B-Tree 的区别**

1）B树的关键字和数据是放在一起的（非叶子节点会存放数据），叶子节点可以看作外部节点，不包含任何信息。B+数的非叶子节点中只含有关键字和指向下一个节点的索引，数据只放在叶子节点中。

2）B树中，越靠近根节点的记录查找得越快，知道找到关键字即可确定记录的存在; 而B+树中每个节点查找的时间基本是一样的。都需要从根节点走到叶子节点，而且在叶子节点中还要再比较关键字。在实际应用中却是 B+树的性能要好些。因为 B+树的非叶子节点不存放实际的数据,这样每个节点可容纳的元素个数比 B-树多,树高比 B-树小,这样带来的好处是减少磁盘访问次数。

**为什么说B+树比B树更适合实际应用中操作系统的文件索引和数据库索引？**

1）B+树的读写磁盘代价更低

​	B+树的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对 B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中,那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说 IO 读写次数也就降低了。

2）B+树的查询更加稳定

​	由于非终结点并不是最终指向文件内容的结点,而只是叶子结点中关键字的索引。任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同,导致每一个数据的查询效率相当。

##### 2.3、聚簇索引和非聚簇索引

​	聚簇索引并不是一种单独的索引类型,而是一种**数据存储方式**。术语‘聚簇’表示数据行和相邻的键值聚簇的存储在一起。如下图,左侧的索引就是聚簇索引,因为数据行在磁盘的排列和索引排序保持一致。

![image-20210407154814632](/home/lin/桌面/pics/image-20210407154814632.png)

1）聚集索引的好处

* 按照聚簇索引排列顺序，查询显示一定范围数据的时候，由于数据是紧密相连，数据库不用从多个数据块中提取数据，所以节省了大量的IO操作

2）聚集索引的坏处

* 对于 mysql 数据库目前只有 innodb 数据引擎支持聚簇索引,而 Myisam 并不支持聚簇索引。
* 由于数据库物理存簇排序方式只能有一种，所以每个Mysql的表只能有一个聚簇索引，一般就是主键
* 更新聚簇索引列的代价很高，会强制InnoDB将每个被更新的行移动到新的位置。
* 插入新行，移动行的时候可能会产生”页分裂“问题。当行的主键值必须将这一行插入某个已满的页中时。存储引擎就会将该页分为两个页面来进行容纳。会导致表占用更多的磁盘空间

#### 3、MySQL索引的分类

##### 3.1、单值索引

概念：即一个索引只包含单个列，一个表就可以有多个单列索引

语法：

```sql
# 和表一起创建
CREATE TABLE customer (
    id INT(10) UNSIGNED AUTO_INCREMENT,
    customer_no VARCHAR(200),customer_name VARCHAR(200),
	PRIMARY KEY(id),
	KEY (customer_name)
);
```

```sql
# 单独创建单值索引
CREATE INDEX idx_customer_name ON customer(customer_name);
```

##### 3.2、唯一索引

概念：索引列必须唯一，但允许有空值

```sql
# 和表一起创建
CREATE TABLE customer (
    id INT(10) UNSIGNED AUTO_INCREMENT,
    customer_no VARCHAR(200),customer_name VARCHAR(200),
    PRIMARY KEY(id),
    KEY (customer_name),
    UNIQUE (customer_no)
);
```

```java
# 单独建唯一索引:
CREATE UNIQUE INDEX idx_customer_no ON customer(customer_no);
```

##### 3.3、主键索引

概念：设定为主键后数据库会自动建立索引，innodb为聚簇索引

```sql
# 随表一起创建
CREATE TABLE customer (
    id INT(10) UNSIGNED AUTO_INCREMENT,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id)
);
```

```sql
# 单独建主键索引:
ALTER TABLE customer add PRIMARY KEY customer(customer_no);
```

```sql
# 删除主键索引
ALTER TABLE customer drop PRIMARY KEY ;
```

```sql
# 修改主键索引
必须先删除掉(drop)原索引,再新建(add)索引
```

##### 3.4、复合索引

概念：即一个索引包含多个列

```sql
# 随表一起建索引
CREATE TABLE customer (
    id INT(10) UNSIGNED AUTO_INCREMENT,
    customer_no VARCHAR(200),customer_name VARCHAR(200),
    PRIMARY KEY(id),
    KEY (customer_name),
    UNIQUE (customer_name),
    KEY (customer_no,customer_name)
);
```

单独创建索引

```sql
CREATE INDEX idx_no_name ON customer(customer_no,customer_name);
```

#### 4、创建索引的时机

##### 4.1、适合创建索引的情况

* 主动建立唯一索引
* 频繁作为查询条件的字段应该建立索引
* 查询中与其他表关联的字段，外键关系建立索引
* 单键/组合索引的选择问题，**组合索引性价比高**
* 查询中排序的字段,排序字段若通过索引去访问将大大提高排序速度
* 查询中统计或者分组字段

##### 4.2、不适合创建索引的情况

* 表记录太少
* 经常增删改的表或者字段
* where条件里用不到的字段不创建索引
* 过滤性不好的不适合索引

#### 5、单表使用索引常见的索引失效

##### 5.1、全值匹配我最爱

###### 5.1.1、有以下SQL语句

```sql
EXPLAIN SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30
EXPLAIN SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptid=4
EXPLAIN SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptid=4 AND emp.name = 'abcd'
```

###### 5.1.2、建立索引

```sql
create index idx_age_id_name on emp(age,deptid,name);

就相当于创建了三个索引 ： 
	name
	name + status
	name + status + address
```

![image-20210407165348369](/home/lin/桌面/pics/image-20210407165348369.png)



结论：全值匹配我最爱指的是，查询的字段按照顺序在索引中都可以匹配到！

![image-20210407165449029](/home/lin/桌面/pics/image-20210407165449029.png)

SQL 中查询字段的顺序,跟使用索引中字段的顺序,没有关系。优化器会在不影响 SQL 执行结果的前提下,给你自动地优化。

##### 5.2、最佳左前前缀法则

![image-20210407183811537](/home/lin/桌面/pics/image-20210407183811537.png)

查询字段与索引字段顺序的不同会导致,索引无法充分使用,甚至索引失效!

原因:使用复合索引,需要遵循最佳左前缀法则,即如果索引了多列,要遵守最左前缀法则。**指的是查询从索引的最左前列开始并且不跳过索引中的列**。

**结论:过滤条件要使用索引必须按照索引建立时的顺序,依次满足,一旦跳过某个字段,索引后面的字段都无法被使用。**

##### 5.3、不要在索引列上做计算

不在索引列上做任何操作(计算、函数、(自动 or 手动)类型转换),会导致索引失效而转向全表扫描。

* 在查询列上使用了函数**（等号左边无计算）**

  ```sql
  EXPLAIN SELECT SQL_NO_CACHE * FROM emp WHERE age=30;
  EXPLAIN SELECT SQL_NO_CACHE * FROM emp WHERE LEFT(age,3)=30;
  ```

  ![image-20210408173605235](/home/lin/桌面/pics/image-20210408173605235.png)

* 在查询列上做了转换

  ```sql
  create index idx_name on emp(name);
  explain select sql_no_cache * from emp where name='30000';
  explain select sql_no_cache * from emp where name=30000;
  ```

  注：字符串不加单引号，则会在name列上做一次转换！

  ![image-20210408173822897](/home/lin/桌面/pics/image-20210408173822897.png)

##### 5.4、索引列上不能有范围查询

```sql
explain SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptid=5 AND emp.name = 'abcd';
explain SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptid<=5 AND emp.name = 'abcd';
```

![image-20210408174006460](/home/lin/桌面/pics/image-20210408174006460.png)

##### 5.5、尽量用覆盖索引

即查询列和索引列一直,不要写 select *!

```sql
explain SELECT SQL_NO_CACHE * FROM emp WHERE emp.age=30 and deptId=4 and name='XamgXt';
explain SELECT SQL_NO_CACHE age,deptId,name and deptId=4 and name='XamgXt';
```

![image-20210408174334109](/home/lin/桌面/pics/image-20210408174334109.png)

##### 5.6、使用不等于（ !=或者<>）的时候

`mysql` 在使用不等于(!= 或者<>)时,有时会无法使用索引会导致全表扫描。

![image-20210408174606302](/home/lin/桌面/pics/image-20210408174606302.png)

##### 5.7、字段是is not null和is null

![image-20210408174730391](/home/lin/桌面/pics/image-20210408174730391.png)

当字段允许为Null的条件下：

![image-20210408174754446](/home/lin/桌面/pics/image-20210408174754446.png)

**is not null 用不到索引,is null 可以用到索引。**

##### 5.8、like的前后模糊匹配（%）=>前缀不能出现模糊匹配

![image-20210408174924717](/home/lin/桌面/pics/image-20210408174924717.png)

##### 5.9、减少使用or

![image-20210408175011314](/home/lin/桌面/pics/image-20210408175011314.png)

使用union all 或者union来代替

![image-20210408175038957](/home/lin/桌面/pics/image-20210408175038957.png)

##### 5.10、口诀

```java
全职匹配我最爱,最左前缀要遵守;
带头大哥不能死,中间兄弟不能断;
索引列上少计算,范围之后全失效;
LIKE百分写最右,覆盖索引不写*;
不等空值还有OR,索引影响要注意;
VAR引号不可丢,SQL优化有诀窍。
```

#### 6、关联查询优化

##### 6.1、left join

```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

结论:

* 在优化关联查询时,只有在被驱动表上建立索引才有效!
* left join 时,左侧的为驱动表,右侧为被驱动表!

##### 6.2、inner join

结论:

* inner join 时,mysql 会自己帮你把**小结果集**的表选为驱动表。
* straight_join: 效果和 inner join 一样,但是会强制将左侧作为驱动表!

```sql
EXPLAIN SELECT * FROM book inner join class on class.card=book.card;
```

##### 6.3、案例

* 子查询尽量不要放在被驱动表,有可能使用不到索引;
* left join时,尽量让**实体表**作为被驱动表。

能够直接多表关联的尽量直接关联,不用子查询!

#### 7、子查询优化

##### 案例

取所有不为掌门人的员工，按年龄分组！

```sql
select age as '年龄', count(*) as '人数'
from t_emp
where id not in
	(
        select ceo
        from t_dept
        where ceo is not null
    )
group by age;
```

![image-20210408150654607](/home/lin/桌面/pics/image-20210408150654607.png)

如何优化？

* 解决dept表的全表扫描，建立ceo字段的索引

  ```sql
  create index idx_ceo on t_dept(ceo)
  ```

  ![image-20210408150710889](/home/lin/桌面/pics/image-20210408150710889.png)

  ![image-20210408152201991](/home/lin/桌面/pics/image-20210408152201991.png)

* 进一步优化，替换not in

  ```sql
  select age as '年龄', count(*) as '人数'
  from t_emp e left join t_dept d on e.id = d.ceo where d.id is null group by age;
  ```

  ![image-20210408152147380](/home/lin/桌面/pics/image-20210408152147380.png)

#### **8 、排序分组优化**

**where 条件和 on 的判断这些过滤条件,作为优先优化的部门,是要被先考虑的**!其次,如果有分组和排序,那么也要考虑 grouo by 和 order by。

##### 8.1、无过滤不索引

```sql
create index idx_age_deptid_name on emp(age,deptid,name)
explain select * from emp where age=40 order by deptid;
explain select * from emp order by age,deptid;
explain select * from emp order by age,deptid limit 10;
```

![image-20210408153321884](/home/lin/桌面/pics/image-20210408153321884.png)

![image-20210408153351726](/home/lin/桌面/pics/image-20210408153351726.png)

using filesort 说明进行了手工排序!原因在于没有 where 作为过滤条件!

![image-20210408153421245](/home/lin/桌面/pics/image-20210408153421245.png)

结论: 无过滤,不索引。**where,limt 都相当于一种过滤条件,所以才能使用上索引**!

##### 8.2、顺序错，必排序

```sql
explain select * from emp where age=45 order by deptid,name;
```

![image-20210408153855093](/home/lin/桌面/pics/image-20210408153855093.png)

```sql
explain select * from emp where age=45 order by
deptid,empno;
-- empno 字段并没有建立索引,因此也无法用到索引,此字段需要排序
```

![image-20210408153919661](/home/lin/桌面/pics/image-20210408153919661.png)

```sql
explain select * from emp where age=45 order by name,deptid;
-- where 两侧列的顺序可以变换,效果相同,但是 order by 列的顺序不能随便变换
```

![image-20210408154118458](/home/lin/桌面/pics/image-20210408154118458.png)

```sql
explain select * from emp where deptid=45 order by age;
-- deptid 作为过滤条件的字段,无法使用索引,因此排序没法用上索引
```

![image-20210408160345815](/home/lin/桌面/pics/image-20210408160345815.png)

##### 8.3、方向反必排序

```sql
explain select * from emp where age=45 order by deptid desc, name desc ;
--如果可以用上索引的字段都使用正序或者逆序,实际上是没有任何影响的,无非将结果集调换顺序。
```

![image-20210408160606426](/home/lin/桌面/pics/image-20210408160606426.png)

如果可以用上索引的字段都使用正序或者逆序,实际上是没有任何影响的,无非将结果集调换顺序。

```sql
explain select * from emp where age=45 order by deptid asc, name desc;
--如果排序的字段,顺序有差异,就需要将差异的部分,进行一次倒置顺序,因此还是需要手动排序的!
```

![image-20210408160703553](/home/lin/桌面/pics/image-20210408160703553.png)

##### 8.4、索引的选择

例1：

```sql
explain SELECT SQL_NO_CACHE * FROM emp WHERE age =30 AND empno <101000 ORDER BY NAME ;
```

索引建立：

第一种：建立三个字段的复合索引

```sql
create index idx_age_empno_name on emp(age,empno,name);
--会发现using filesort依然存在
```

![image-20210408161320875](/home/lin/桌面/pics/image-20210408161320875.png)

优化：三个字段的符合索引,没有意义,因为 empno 和 name 字段只能选择其一!

```sql
-- 方法一
create index idx_age_name on emp(age,name);
-- 方法二：比较好
create index idx_age_empno on emp(age,empno);
```

![image-20210408163040606](/home/lin/桌面/pics/image-20210408163040606.png)

结论: 当范围条件和 group by 或者 order by的字段出现二选一时 ,优先观察条件字段的过滤数量,如果过滤的数据足够多,而需要排序的数据并不多时,优先把索引放在范围字段上。反之,亦然。

##### 8.5、`using filesort`

###### 8.5.1、`MySQL`的排序算法

1）双路排序：

	MySQL 4.1 之前是使用双路排序,字面意思就是两次扫描磁盘,最终得到数据,读取行指针和 orderby 列,对他们进行排序,然后扫描已经排序好的列表,按照列表中的值重新从列表中读取对应的数据输出。

2）单路排序

```
从磁盘读取查询需要的所有列,按照 order by 列在 buffer 对它们进行排序,然后扫描排序后的列表进行输出,它的效率更快一些,避免了第二次读取数据。并且把随机 IO 变成了顺序 IO,但是它会使用更多的空间,
因为它把每一行都保存在内存中了
```

单路排序问题：在 sort_buffer 中,方法 B 比方法 A 要多占用很多空间,因为方法 B 是把所有字段都取出, 所以有可能取出的数据的总大小超出了 sort_buffer 的容量,导致每次只能取 sort_buffer 容量大小的数据,进行排序(创建 tmp 文件,多路合并),排完再取取 sort_buffer 容量大小,再排......从而多次 I/O。

###### 8.5.2、如何优化

1）增大sort_buffer_size参数的设置

2）增大 max_length_for_sort_data 参数的设置

3）减少 select 后面的查询的字段

##### 8.6、使用覆盖索引

覆盖索引:`SQL`只需要通过索引就可以返回查询所需要的数据,而不必通过二级索引查到主键之后再去查询数据。

![image-20210408164204991](/home/lin/桌面/pics/image-20210408164204991.png)

##### 8.7、group by

group by 使用索引的原则几乎跟 order by 一致 ,唯一区别是 groupby 即使没有过滤条件用到索引,也可以直接使用索引。

![image-20210408164324135](/home/lin/桌面/pics/image-20210408164324135.png)

### 六、`SQL`优化

#### 1、SQL优化的步骤

##### 1.1、查看SQL执行频率

MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息。

下面的命令显示了当前session中所有统计参数的值：

```cmd
show status like 'Com_______';
```

![1552487172501](file:///home/lin/桌面/netty-study/assets/1552487172501.png?lastModify=1617622122)

```cmd
show status like 'Innodb_rows_%';
```

![1552487245859](file:///home/lin/桌面/netty-study/assets/1552487245859.png?lastModify=1617622166)

Com_xxx 表示每个 xxx 语句执行的次数，我们通常比较关心的是以下几个统计参数。

| 参数                 | 含义                                                         |
| :------------------- | ------------------------------------------------------------ |
| Com_select           | 执行 select 操作的次数，一次查询只累加 1。                   |
| Com_insert           | 执行 INSERT 操作的次数，对于批量插入的 INSERT 操作，只累加一次。 |
| Com_update           | 执行 UPDATE 操作的次数。                                     |
| Com_delete           | 执行 DELETE 操作的次数。                                     |
| Innodb_rows_read     | select 查询返回的行数。                                      |
| Innodb_rows_inserted | 执行 INSERT 操作插入的行数。                                 |
| Innodb_rows_updated  | 执行 UPDATE 操作更新的行数。                                 |
| Innodb_rows_deleted  | 执行 DELETE 操作删除的行数。                                 |
| Connections          | 试图连接 MySQL 服务器的次数。                                |
| Uptime               | 服务器工作时间。                                             |
| Slow_queries         | 慢查询的次数。                                               |

##### 1.2、定位低效率执行SQL

可以通过以下两种方式定位执行效率较低的 SQL 语句。

* 慢查询日志 : 通过慢查询日志定位那些执行效率较低的 SQL 语句，用--log-slow-queries[=file_name]选项启动时，mysqld 写一个包含所有执行时间超过 long_query_time 秒的 SQL 语句的日志文件。
* show processlist  : 慢查询日志在查询结束以后才纪录，所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用show processlist命令查看当前MySQL在进行的线程，包括线程的状态、是否锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化。

![1556098544349](file:///home/lin/桌面/netty-study/assets/1556098544349.png?lastModify=1617622335)

```txt
1） id列，用户登录mysql时，系统分配的"connection_id"，可以使用函数connection_id()查看

2） user列，显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句

3） host列，显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户

4） db列，显示这个进程目前连接的是哪个数据库

5） command列，显示当前连接的执行的命令，一般取值为休眠（sleep），查询（query），连接（connect）等

6） time列，显示这个状态持续的时间，单位是秒

7） state列，显示使用当前连接的sql语句的状态，很重要的列。state描述的是语句执行中的某一个状态。一个sql语句，以查询为例，可能需要经过copying to tmp table、sorting result、sending data等状态才可以完成

8） info列，显示这个sql语句，是判断问题语句的一个重要依据
```

##### 1.3、explain分析执行计划（重点）

通过以上步骤查询到效率低的 SQL 语句后，可以通过 EXPLAIN或者 DESC命令获取 MySQL如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。

查询SQL语句的执行计划

```sql
explain  select * from tb_item where id = 1;
```

![1552487489859](file:///home/lin/桌面/netty-study/assets/1552487489859.png?lastModify=1617622558)

| 字段          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。 |
| select_type   | 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（子查询中的第一个 SELECT）等 |
| table         | 输出结果集的表                                               |
| type          | 表示表的连接类型，性能由好到差的连接类型为( system  --->  const  ----->  eq_ref  ------>  ref  ------->  ref_or_null---->  index_merge  --->  index_subquery  ----->  range  ----->  index  ------> all ) |
| possible_keys | 表示查询时，可能使用的索引                                   |
| key           | 表示实际使用的索引                                           |
| key_len       | 索引字段的长度                                               |
| rows          | 扫描行的数量                                                 |
| extra         | 执行情况的说明和描述                                         |

###### 1.3.1、explain之id（重要）

id 字段是 select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。id 情况有三种 ：

1）**id 相同**表示加载表的顺序是从上到下。

2）**id 不同**id值越大，优先级越高，越先被执行。 

3）**id 有相同，也有不同，同时存在**。id相同的可以认为是一组，从上往下顺序执行；在所有的组中，id的值越大，优先级越高，越先执行。

###### 1.3.2、explain 之 select_type

表示 SELECT 的类型，常见的取值，如下表所示：

| select_type  | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| SIMPLE       | 简单的select查询，查询中不包含子查询或者UNION                |
| PRIMARY      | 查询中若包含任何复杂的子查询，最外层查询标记为该标识         |
| SUBQUERY     | 在SELECT 或 WHERE 列表中包含了子查询                         |
| DERIVED      | 在FROM 列表中包含的子查询，被标记为 DERIVED（衍生） MYSQL会递归执行这些子查询，把结果放在临时表中 |
| UNION        | 若第二个SELECT出现在UNION之后，则标记为UNION ； 若UNION包含在FROM子句的子查询中，外层SELECT将被标记为 ： DERIVED |
| UNION RESULT | 从UNION表获取结果的SELECT                                    |

###### 1.3.3、explain 之 table

展示这一行的数据是关于哪一张表的 

###### 1.3.4、explain之type（重要）

type 显示的是访问类型，是较为**重要**的一个指标，可取值为：

| type       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| NULL       | MySQL不访问任何表，索引，直接返回结果                        |
| system     | 表只有一行记录(等于系统表)，这是const类型的特例，一般不会出现 |
| const      | 表示通过索引一次就找到了，const 用于比较primary key 或者 unique 索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL 就能将该查询转换为一个常亮。const于将 "主键" 或 "唯一" 索引的所有部分与常量值进行比较 |
| **eq_ref** | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，**关联查询出的记录只有一条**。常见于主键或唯一索引扫描 |
| **ref**    | 非唯一性索引扫描，返回**匹配某个单独值**的所有行。本质上也是一种索引访问，返回所有匹配某个单独值的所有行（多个） |
| **range**  | **只检索给定返回的行**，使用一个索引来选择行。 where 之后出现 between ， < , > , in 等操作。 |
| **index**  | index 与 ALL的区别为  **index 类型只是遍历了索引树，** 通常比ALL 快， ALL 是遍历数据文件。 |
| **all**    | 将遍历全表以找到匹配的行                                     |

结果值从最好到最坏依次是

```
NULL > system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

system > const > eq_ref > ref > range > index > ALL
```

<span style="color:#ff0000">注：一般来说我们需要保证查询至少达到range级别，最好是达到ref</span>

###### 1.3.5、explain之key

```
possible_keys : 显示可能应用在这张表的索引， 一个或多个。 

key ： 实际使用的索引， 如果为NULL， 则没有使用索引。

key_len : 表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下， 长度越短越好 。
```

###### 1.3.6、explain之rows

扫描行的数量

###### 1.3.7、explain之extra

其他的额外的执行计划信息，在该列展示

| extra            | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| using  filesort  | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称为 “文件排序”, **效率低。** |
| using  temporary | 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于 order by 和 group by； **效率低** |
| using  index     | 表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。 |

##### 1.4、show profile分析SQL

Mysql从5.0.37版本开始增加了对 show profiles 和 show profile 语句的支持。show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。

通过 have_profiling 参数，能够看到当前MySQL是否支持profile：

![1552488401999](file:///home/lin/桌面/netty-study/assets/1552488401999.png?lastModify=1617623920)

默认profiling是关闭的，可以通过set语句在Session级别开启profiling：

```sql
set profiling=1; //开启profiling 开关；
```

通过profile，我们能够更清楚地了解SQL执行的过程。

首先，我们可以执行一系列的操作，如下图所示：

```sql
show databases;

use db01;

show tables;

select * from tb_item where id < 5;

select count(*) from tb_item;
```

执行完上述命令后，再执行show profiles 指令， 来查看SQL语句执行的耗时：

![1552489017940](file:///home/lin/桌面/netty-study/assets/1552489017940.png?lastModify=1617624013)

通过`show profile for query query_id `语句可以查看到该SQL执行过程中每个线程的状态和消耗的时间：

![1552489053763](file:///home/lin/桌面/netty-study/assets/1552489053763.png?lastModify=1617624050)

```tex
TIP ：
	Sending data 状态表示MySQL线程开始访问数据行并把结果返回给客户端，而不仅仅是返回个客户端。由于在Sending data状态下，MySQL线程往往需要做大量的磁盘读取操作，所以经常是整各查询中耗时最长的状态。
```

在获取到最消耗时间的线程状态后，MySQL支持进一步选择all、cpu、block io 、context switch、page faults等明细类型类查看MySQL在使用什么资源上耗费了过高的时间。例如，选择查看CPU的耗费时间  ：

![1552489671119](file:///home/lin/桌面/netty-study/assets/1552489671119.png?lastModify=1617624108)

| 字段       | 含义                           |
| ---------- | ------------------------------ |
| Status     | sql 语句执行的状态             |
| Duration   | sql 执行过程中每一个步骤的耗时 |
| CPU_user   | 当前用户占有的cpu              |
| CPU_system | 系统占有的cpu                  |

#### 2、优化方法

##### 2.1、大批量插入数据（插入sql.log）

当使用load命令导入数据的时候，适当的设置可以提高导入的效率

![1556269346488](file:///home/lin/桌面/netty-study/assets/1556269346488.png?lastModify=1617674378)

对于`InnoDB`类型的表，有以下几种方式可以提高导入的效率

**1）主键顺序插入**

因为`InnoDB`类型的表是按照主键的顺序保存的，所以将导入的数据按照主键的顺序排列，可以有效的提高导入数据的效率。如果`InnoDB`表没有主键，那么系统会自动默认创建一个内部列作为主键，所以如果可以给表创建一个主键，将可以利用这点，来提高导入数据的效率。

**2）关闭唯一性检验**

在导入数据前执行 SET UNIQUE_CHECKS=0，关闭唯一性校验，在导入结束后执行SET UNIQUE_CHECKS=1，恢复唯一性校验，可以提高导入的效率。

**3）手动提交事务**

如果应用使用自动提交的方式，建议在导入前执行 SET `AUTOCOMMIT=0`，关闭自动提交，导入结束后再执行 `SET AUTOCOMMIT=1`，打开自动提交，也可以提高导入的效率

##### 2.2、优化insert语句

* 如果需要同时对一张表插入很多行数据时，应该尽量使用多个值表的insert语句，这种方式将大大的缩减客户端与数据库之间的连接、关闭等消耗。使得效率比分开执行的单个insert语句快。

  示例， 原始方式为：

  ```sql
  insert into tb_test values(1,'Tom');
  insert into tb_test values(2,'Cat');
  insert into tb_test values(3,'Jerry');
  ```

  优化

  ```sql
  insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
  ```

* 在事务中进行数据插入

  ```sql
  start transaction;
  insert into tb_test values(1,'Tom');
  insert into tb_test values(2,'Cat');
  insert into tb_test values(3,'Jerry');
  commit;
  ```

* 数据有序插入

  ```sql
  insert into tb_test values(4,'Tim');
  insert into tb_test values(1,'Tom');
  insert into tb_test values(3,'Jerry');
  insert into tb_test values(5,'Rose');
  insert into tb_test values(2,'Cat');
  ```

  优化后

  ```sql
  insert into tb_test values(1,&#39;Tom&#39;);
  insert into tb_test values(2,&#39;Cat&#39;);
  insert into tb_test values(3,&#39;Jerry&#39;);
  insert into tb_test values(4,&#39;Tim&#39;);
  insert into tb_test values(5,&#39;Rose&#39;);与
  ```

* 优化order by语句

  * 两种排序方式

    1）通过对返回数据进行排序，也就是通常说的filesort排序，所有不是通过索引直接返回排序结果的排序都叫filesort排序

    ![1556335817763](file:///home/lin/桌面/netty-study/assets/1556335817763.png?lastModify=1617674872)

    2）通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

    ![1556335866539](file:///home/lin/桌面/netty-study/assets/1556335866539.png?lastModify=1617674900)

    多字段排序

    ![1556336352061](file:///home/lin/桌面/netty-study/assets/1556336352061.png?lastModify=1617674991)

    了解了MySQL的排序方式，优化目标就清晰了：**尽量减少额外的排序，通过索引直接返回有序数据**。**where 条件和Order by 使用相同的索引，并且Order By 的顺序和索引顺序相同**， 并且Order  by 的字段都是升序，或者都是降序。否则肯定需要额外的操作，这样就会出现FileSort。

* Filesort 的优化

  通过创建合适的索引，能够减少 Filesort 的出现，但是在某些情况下，条件限制不能让Filesort消失，那就需要加快 Filesort的排序操作。对于Filesort ， MySQL 有两种排序算法：

  1）两次扫描算法 ：MySQL4.1 之前，使用该方式排序。首先根据条件取出排序字段和行指针信息，然后在排序区 sort buffer 中排序，如果sort buffer不够，则在临时表 temporary table 中存储排序结果。完成排序之后，再根据行指针回表读取记录，该操作可能会导致大量随机I/O操作。

  2）一次扫描算法：一次性取出满足条件的所有字段，然后在排序区 sort  buffer 中排序后直接输出结果集。排序时内存开销较大，但是排序效率比两次扫描算法要高。

  

  MySQL 通过比较系统变量 max_length_for_sort_data 的大小和Query语句取出的字段总大小， 来判定是否那种排序算法，如果max_length_for_sort_data 更大，那么使用第二种优化之后的算法；否则使用第一种。

  可以适当提高 sort_buffer_size  和 max_length_for_sort_data  系统变量，来增大排序区的大小，提高排序的效率。

  ![1556338367593](file:///home/lin/桌面/netty-study/assets/1556338367593.png?lastModify=1617675376)

##### 2.3、优化group by 语句

**由于GROUP BY 实际上也同样会进行排序操作，而且与ORDER BY 相比，GROUP BY 主要只是多了排序之后的分组操作。**当然，如果在分组的时候还使用了其他的一些聚合函数，那么还需要一些聚合函数的计算。所以，在GROUP BY 的实现过程中，与 ORDER BY 一样也可以利用到索引。

如果查询包含group by但是用户想要避免排序结果的消耗，则可以执行order by null禁止排序

```sql
drop index idx_emp_age_salary on emp;

explain select age,count(*) from emp group by age;
```

![1556339573979](file:///home/lin/桌面/netty-study/assets/1556339573979.png?lastModify=1617675711)

优化后

```sql
explain select age,count(*) from emp group by age order by null;
```

![1556339633161](file:///home/lin/桌面/netty-study/assets/1556339633161.png?lastModify=1617675741)

从上面的例子可以看出，第一个SQL语句需要进行"filesort"，而第二个SQL由于order  by  null 不需要进行 "filesort"， 而上文提过Filesort往往非常耗费时间。

创建索引 ：

```java
create index idx_emp_age_salary on emp(age,salary)；
```

![1556339688158](file:///home/lin/桌面/netty-study/assets/1556339688158.png?lastModify=1617675774)

##### 2.4、优化嵌套查询

Mysql4.1版本之后，开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，**有些情况下，子查询是可以被更高效的连接（JOIN）替代。**

示例 ，查找有角色的所有的用户信息 : 

```sql
explain select * from t_user where id in (select user_id from user_role );
```

执行计划为：

![1556359399199](file:///home/lin/桌面/netty-study/assets/1556359399199.png?lastModify=1617675918)

优化后

```sql
explain select * from t_user u , user_role ur where u.id = ur.user_id;
```

![1556359482142](file:///home/lin/桌面/netty-study/assets/1556359482142.png?lastModify=1617675959)

**连接(Join)查询之所以更有效率一些 ，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上需要两个步骤的查询工作。**

#### 3、优化OR条件

对于包含OR的查询子句，如果要利用索引，则OR之间的每个条件列都必须用到索引 ， 而且不能使用到复合索引； 如果没有索引，则应该考虑增加索引。

获取 emp 表中的所有的索引 ： 

![1556354464657](file:///home/lin/桌面/netty-study/assets/1556354464657.png?lastModify=1617676411)

示例 ： 

```SQL
explain select * from emp where id = 1 or age = 30;
```

![1556354887509](/home/lin/桌面/netty-study/assets/1556354887509.png)

![1556354920964](/home/lin/桌面/netty-study/assets/1556354920964.png)  

建议使用 union 替换 or ： 

![1556355027728](/home/lin/桌面/netty-study/assets/1556355027728.png) 

我们来比较下重要指标，发现主要差别是 type 和 ref 这两项

type 显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：

```
system > const > eq_ref > ref > fulltext > ref_or_null  > index_merge > unique_subquery > index_subquery > range > index > ALL
```

UNION 语句的 type 值为 ref，OR 语句的 type 值为 range，可以看到这是一个很明显的差距

UNION 语句的 ref 值为 const，OR 语句的 type 值为 null，const 表示是常量值引用，非常快

这两项的差距就说明了 UNION 要优于 OR 。

#### 4、优化分页查询

一般分页查询时，通过创建覆盖索引能够比较好地提高性能。一个常见又非常头疼的问题就是 limit 2000000,10  ，此时需要MySQL排序前2000010 记录，仅仅返回2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 。

![1556361314783](/home/lin/桌面/netty-study/assets/1556361314783.png) 

##### 4.1、优化思路一

在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。

![1556416102800](/home/lin/桌面/netty-study/assets/1556416102800.png) 



##### 4.2、优化思路二

该方案适用于主键自增的表，可以把Limit 查询转换成某个位置的查询 。

![1556363928151](/home/lin/桌面/netty-study/assets/1556363928151.png) 

#### 5、使用SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

##### 5.1、USE INDEX

在查询语句中表名的后面，添加 use index 来提供希望MySQL去参考的索引列表，就可以让MySQL不再考虑其他可用的索引。

```
create index idx_seller_name on tb_seller(name);
```

![1556370971576](/home/lin/桌面/netty-study/assets/1556370971576.png) 

##### 5.2、IGNORE INDEX

如果用户只是单纯的想让MySQL忽略一个或者多个索引，则可以使用 ignore index 作为 hint 。

```
 explain select * from tb_seller ignore index(idx_seller_name) where name = '小米科技';
```

![1556371004594](/home/lin/桌面/netty-study/assets/1556371004594.png) 

##### 5.3、FORCE INDEX

为强制MySQL使用一个特定的索引，可在查询中使用 force index 作为hint 。 

``` SQL
create index idx_seller_address on tb_seller(address);
```

![1556371355788](/home/lin/桌面/netty-study/assets/1556371355788.png) 



