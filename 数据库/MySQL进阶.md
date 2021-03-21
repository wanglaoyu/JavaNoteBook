# 事务

## 一、四大性质ACID

分别是原子性、一致性、隔离性、持久性。

### 1、原子性（Atomicity）

原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响。

### 2、一致性（Consistency）

一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。举例来说，假设用户A和用户B两者的钱加起来一共是1000，那么不管A和B之间如何转账、转几次账，事务结束后两个用户的钱相加起来应该还得是1000，这就是事务的一致性。其他三个性质目的是保持一致性

### 3、隔离性（Isolation）

隔离性是当多个用户并发访问数据库时，比如同时操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。关于事务的隔离性数据库提供了多种隔离级别，稍后会介绍到。

### 4、持久性（Durability）

持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。例如我们在使用JDBC操作数据库时，在提交事务方法后，提示用户事务操作完成，当我们程序执行完成直到看到提示后，就可以认定事务已经正确提交，即使这时候数据库出现了问题，也必须要将我们的事务完全执行完成。否则的话就会造成我们虽然看到提示事务处理完毕，但是数据库因为故障而没有执行事务的重大错误。这是不允许的。

## 二、事务的隔离级别



### 1、为什么要设置隔离级别

在数据库操作中，在并发的情况下可能出现如下问题：

- 更新丢失（Lost update） 
  如果多个线程操作，基于同一个查询结构对表中的记录进行修改，那么后修改的记录将会覆盖前面修改的记录，前面的修改就丢失掉了，这就叫做更新丢失。这是因为系统没有执行任何的锁操作，因此并发事务并没有被隔离开来。 
  第1类丢失更新：事务A撤销时，把已经提交的事务B的更新数据覆盖了。 
  ![这里写图片描述](https://img-blog.csdn.net/20170725154312638) 
  第2类丢失更新：事务A覆盖事务B已经提交的数据，造成事务B所做的操作丢失。 
  ![这里写图片描述](https://img-blog.csdn.net/20170725154357546) 
  解决方法：对行加锁，只允许并发一个更新事务。
- 脏读（Dirty Reads） 
  脏读（Dirty Read）：A事务读取B事务尚未提交的数据并在此基础上操作，而B事务执行回滚，那么A读取到的数据就是脏数据。 
  ![这里写图片描述](https://img-blog.csdn.net/20170725154520750) 
  解决办法：如果在第一个事务提交前，任何其他事务不可读取其修改过的值，则可以避免该问题。
- 不可重复读（Non-repeatable Reads） 
  一个事务对同一行数据重复读取两次，但是却得到了不同的结果。事务T1读取某一数据后，事务T2对其做了修改，当事务T1再次读该数据时得到与前一次不同的值。 
  ![这里写图片描述](https://img-blog.csdn.net/20170725154610970) 
  解决办法：如果只有在修改事务完全提交之后才可以读取数据，则可以避免该问题。
- 幻象读 
  指两次执行同一条 select  语句会出现不同的结果，第二次读会增加一数据行，并没有说这两次执行是在同一个事务中。一般情况下，幻象读应该正是我们所需要的。但有时候却不是，如果打开的游标，在对游标进行操作时，并不希望新增的记录加到游标命中的数据集中来。隔离级别为 游标稳定性  的，可以阻止幻象读。例如：目前工资为1000的员工有10人。那么事务1中读取所有工资为1000的员工，得到了10条记录；这时事务2向员工表插入了一条员工记录，工资也为1000；那么事务1再次读取所有工资为1000的员工共读取到了11条记录。 
  ![这里写图片描述](https://img-blog.csdn.net/20170725154713322) 
  解决办法：如果在操作事务完成数据处理之前，任何其他事务都不可以添加新数据，则可避免该问题。







### 不可重复读和幻读的区别

 当然,  从总的结果来看,  似乎两者都表现为两次读取的结果不一致.
 但如果你从控制的角度来看,  两者的区别就比较大
 对于前者,  只需要锁住满足条件的记录
 对于后者,  要锁住满足条件及其相近的记录

\-----------------------------------------------------------

我这么理解是否可以？
 避免不可重复读需要锁行就行
 避免幻影读则需要锁表

\------------------------

\####不可重复读和幻读的区别####
很多人容易搞混不可重复读和幻读，确实这两者有些相似。但不可重复读重点在于update和delete，而幻读的重点在于insert。

如果使用锁机制来实现这两种隔离级别，在可重复读中，该sql第一次读取到数据后，就将这些数据加锁，其它事务无法修改这些数据，就可以实现可重复 读了。但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会 发现莫名其妙多了一条之前没有的数据，这就是幻读，不能通过行锁来避免。需要Serializable隔离级别  ，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。

**所以说不可重复读和幻读最大的区别，就在于如何通过锁机制来解决他们产生的问题。**













### 2、事务的隔离级别

数据库事务的隔离级别有4个，由低到高依次为Read uncommitted(未授权读取、读未提交)、Read  committed（授权读取、读提交）、Repeatable  read（可重复读取）、Serializable（序列化），这四个级别可以逐个解决脏读、不可重复读、幻象读这几类问题。

- Read uncommitted(未授权读取、读未提交)： 
  如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。该隔离级别可以通过“排他写锁”实现。这样就避免了更新丢失，却可能出现脏读。也就是说事务B读取到了事务A未提交的数据。
- Read committed（授权读取、读提交）： 
  读取数据的事务允许其他事务继续访问该行数据，但是未提交的写事务将会禁止其他事务访问该行。该隔离级别避免了脏读，但是却可能出现不可重复读。事务A事先读取了数据，事务B紧接了更新了数据，并提交了事务，而事务A再次读取该数据时，数据已经发生了改变。
- Repeatable read（可重复读取）： 
  可重复读是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，即使第二个事务对数据进行修改，第一个事务两次读到的的数据是一样的。这样就发生了在一个事务内两次读到的数据是一样的，因此称为是可重复读。读取数据的事务将会禁止写事务（但允许读事务），写事务则禁止任何其他事务。这样避免了不可重复读取和脏读，但是有时可能出现幻象读。（读取数据的事务）这可以通过“共享读锁”和“排他写锁”实现。
- Serializable（序列化）： 
  提供严格的事务隔离。它要求事务序列化执行，事务只能一个接着一个地执行，但不能并发执行。如果仅仅通过“行级锁”是无法实现事务序列化的，必须通过其他机制保证新插入的数据不会被刚执行查询操作的事务访问到。序列化是最高的事务隔离级别，同时代价也花费最高，性能很低，一般很少使用，在该级别下，事务顺序执行，不仅可以避免脏读、不可重复读，还避免了幻像读。 
  ![这里写图片描述](https://img-blog.csdn.net/20170725154818659) 
  隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。对于多数应用程序，可以优先考虑把数据库系统的隔离级别设为Read  Committed。它能够避免脏读取，而且具有较好的并发性能。尽管它会导致不可重复读、幻读和第二类丢失更新这些并发问题，在可能出现这类问题的个别场合，可以由应用程序采用悲观锁或乐观锁来控制。大多数数据库的默认级别就是Read committed，比如Sql Server , Oracle。MySQL的默认隔离级别就是Repeatable read。

## 三、悲观锁和乐观锁

虽然数据库的隔离级别可以解决大多数问题，但是灵活度较差，为此又提出了悲观锁和乐观锁的概念。

### 1、悲观锁(事务涉及就会被锁)

悲观锁，它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度。因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制。也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统的数据访问层中实现了加锁机制，也无法保证外部系统不会修改数据。

- 使用场景举例：以MySQL InnoDB为例

商品t_items表中有一个字段status，status为1代表商品未被下单，status为2代表商品已经被下单（此时该商品无法再次下单），那么我们对某个商品下单时必须确保该商品status为1。假设商品的id为1。 
如果不采用锁，那么操作方法如下：

```mysql
//1.查询出商品信息
select status from  t_items where id=1;
//2.根据商品信息生成订单,并插入订单表 t_orders 
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_items set status=2;
```



但是上面这种场景在高并发访问的情况下很可能会出现问题。例如当第一步操作中，查询出来的商品status为1。但是当我们执行第三步Update操作的时候，有可能出现其他人先一步对商品下单把t_items中的status修改为2了，但是我们并不知道数据已经被修改了，这样就可能造成同一个商品被下单2次，使得数据不一致。所以说这种方式是不安全的。

- 使用悲观锁来解决问题

在上面的场景中，商品信息从查询出来到修改，中间有一个处理订单的过程，使用悲观锁的原理就是，当我们在查询出t_items信息后就把当前的数据锁定，直到我们修改完毕后再解锁。那么在这个过程中，因为t_items被锁定了，就不会出现有第三者来对其进行修改了。需要注意的是，要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。我们可以使用命令设置MySQL为非autocommit模式：`set autocommit=0;` 
设置完autocommit后，我们就可以执行我们的正常业务了。具体如下：

```mysql
//0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
//1.查询出商品信息
select status from t_items where id=1 for update;
//2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_items set status=2;
//4.提交事务
commit;/commit work;
```





上面的begin/commit为事务的开始和结束，因为在前一步我们关闭了mysql的autocommit，所以需要手动控制事务的提交。 
上面的第一步我们执行了一次查询操作：`select status from t_items where id=1 for update;`与普通查询不一样的是，我们使用了`select…for update`的方式，这样就通过数据库实现了悲观锁。此时在t_items表中，id为1的那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。需要注意的是，在事务中，只有`SELECT ... FOR UPDATE` 或`LOCK IN SHARE MODE` 操作同一个数据时才会等待其它事务结束后才执行，一般`SELECT ...` 则不受此影响。拿上面的实例来说，当我执行`select status from t_items where id=1 for update;`后。我在另外的事务中如果再次执行`select status from t_items where id=1 for update;`则第二个事务会一直等待第一个事务的提交，此时第二个查询处于阻塞的状态，但是如果我是在第二个事务中执行`select status from t_items where id=1;`则能正常查询出数据，不会受第一个事务的影响。

- Row Lock与Table Lock

使用`select…for update`会把数据给锁住，不过我们需要注意一些锁的级别，MySQL  InnoDB默认Row-Level Lock，所以只有「明确」地指定主键或者索引，MySQL 才会执行Row lock (只锁住被选取的数据)  ，否则MySQL 将会执行Table Lock (将整个数据表单给锁住)。举例如下： 
1、`select * from t_items where id=1 for update;` 
这条语句明确指定主键（id=1），并且有此数据（id=1的数据存在），则采用row lock。只锁定当前这条数据。 
2、`select * from t_items where id=3 for update;` 
这条语句明确指定主键，但是却查无此数据，此时不会产生lock（没有元数据，又去lock谁呢？）。 
3、`select * from t_items where name='手机' for update;` 
这条语句没有指定数据的主键，那么此时产生table lock，即在当前事务提交前整张数据表的所有字段将无法被查询。 
4、`select * from t_items where id>0 for update;` 或者`select * from t_items where id<>1 for update;`（注：<>在SQL中表示不等于） 
上述两条语句的主键都不明确，也会产生table lock。 
5、`select * from t_items where status=1 for update;`（假设为status字段添加了索引） 
这条语句明确指定了索引，并且有此数据，则产生row lock。 
6、`select * from t_items where status=3 for update;`（假设为status字段添加了索引） 
这条语句明确指定索引，但是根据索引查无此数据，也就不会产生lock。

- 悲观锁小结 
  悲观锁并不是适用于任何场景，它也有它存在的一些不足，因为悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。如果加锁的时间过长，其他用户长时间无法访问，影响了程序的并发访问性，同时这样对数据库性能开销影响也很大，特别是对长事务而言，这样的开销往往无法承受。所以与悲观锁相对的，我们有了乐观锁。

### 2、乐观锁（提交修改时才检验冲突）

乐观锁（ Optimistic Locking ）  相对悲观锁而言，乐观锁假设认为数据一般情况下不会造成冲突，所以只会在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则返回用户错误的信息，让用户决定如何去做。实现乐观锁一般来说有以下2种方式：

- - 使用版本号 
    使用数据版本（Version）记录机制实现，这是乐观锁最常用的一种实现方式。何谓数据版本？即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version”  字段来实现。当读取数据时，将version字段的值一同读出，数据每更新一次，对此version值加一。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等，则予以更新，否则认为是过期数据。
  - 使用时间戳 
    乐观锁定的第二种实现方式和第一种差不多，同样是在需要乐观锁控制的table中增加一个字段，名称无所谓，字段类型使用时间戳（timestamp）,  和上面的version类似，也是在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则OK，否则就是版本冲突。









## 四、死锁

### 1.死锁的产生原因

### 2.死锁的解决办法









## 五、MySQL中的事务相关

1自动提交

设置隔离级别

为什么事务中不能混用储存引擎

### 隐式显示锁定 

SELECT ...  LOCK IN  SHARE MODE

SELECT ...   FOR  UPDATE





## 六、多版本并发控制MVCC

MVCC是如何工作的









LOCK TABLES  UNLOCK TABLES  服务器层实现，与搜索引擎无关 









InnoDB  间隙锁（next-key locking）策略防止幻读的出现，间隙锁使得InnoDB不仅仅锁定查询涉及的行，还会对索引中的空隙进行锁定，防止幻影行的插入.





# Mysql逻辑架构

Mysql和其他数据库相比，Mysql有些与众不同，它的架构可以在多种不同场景下中发挥良好作用，主要体现在存储引擎上，插件式的存储架构将查询处理和其他子任务的系统任务和数据的存储提取相分离。	

架构图

![image-20210317113159396](C:\Users\wangdingjian\AppData\Roaming\Typora\typora-user-images\image-20210317113159396.png)



![img](https://img2018.cnblogs.com/blog/1454031/201904/1454031-20190410222744803-621774735.png)

1.连接层

2.服务层

**Management Serveices & Utilities:**系统管理和控制工具

**SQL Interface:** SQL接口   分析器
接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用

**SQL InterfaceParser:**解析器
SQL命令传递到解析器的时候会被解析器验证和解析。

**Optimizer:**查询优化器。
SQL语句在查询之前会使用查询优化器对查询进行优化。在不改变结果的前提下对SQL语句的执行顺序进行优化。
用一个例子就可以理解: select uid,name from user where gender= 1;优化器来决定先投影还是先过滤。
**Cache和Buffer:**查询缓存。
如果查询缓存有命中的查询结果，可以直接去查询缓存中取数据。
这个缓存机制是由一系列小缓存组成比如表缓存，记录缓存，key缓存，权限缓存等

**缓存 和缓冲的区别：**

​	缓存用于读取时 缓冲用于写入时。



3.引擎层

4.存储层

# Mysql的储存引擎

查看命令

```mysql
show engines;
```



## **MyISAM引擎和InnoDB引擎简单对比**

|          | MyISAM引擎                                           | InnoDB引擎                                                   |
| -------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| 主外键   | 不支持                                               | 支持                                                         |
| 事务     | 不支持                                               | 支持                                                         |
| 行表锁   | 表锁。操作一条记录锁住整个表，不适合高并发           | 行锁.使用间隙锁锁住一条记录（及周围的索引），适合高并发，但表锁会出现死锁。 |
| 缓存     | 只缓存索引，不缓存真实数据，每次查询必须打开表来查询 | 缓存索引和真实数据，对内存要求比较高，内存大小对性能有影响   |
| 表空间   | 小                                                   | 大                                                           |
| 关注点   | 节省资源、消耗小.偏读                                | 事务、并发、更大资源。                                       |
| 默认安装 | 是                                                   | 是                                                           |



 备注：复杂的数据库基本不会设置外键约束，不然会大幅降低效率，删除更新表时 也会受到过多限制。

## 其他引擎

Achieve档案引擎：

只支持select和insert操作，不能删除和修改，比MyISAM表要小75%，比InnoDB表小83%

Blackhole引擎：

没有实现任何存储机制，会丢弃所有插入的数值，只把所有操作计入日志。

CSV引擎：

CSV引擎可以将普通的CSV文件作为MySQL的表来处理，但不支持索引。CSV引擎可以作用数据交换的机制。

Memory引擎：

速度快，存在内存，重启就没。

Federated引擎：

 访问其他Mysql服务器的远程代理，可以简单查询，但效率过低，被禁用。





Percona Server 新建了XtraDB引擎可以完全替代InnoDB，阿里巴巴吧大部分MySQL数据库其实使用的是在Percona原型进行修改的版本。









# Mysql索引优化分析

数据库出现性能降SQL慢、执行时间长、等待时间长等问题时需要进行优化处理

| 问题                     | 处理方式   |
| ------------------------ | ---------- |
| 数据过多                 | 分库分表   |
| 关联了太多的表，太多join | SQL优化    |
| 没有充分利用到索引       | 索引建立   |
| 服务器调优及各个参数     | 调整my.cnf |



## MySQL的7种JOIN

| 连接方式                                       |                             语句                             | 效果               |
| ---------------------------------------------- | :----------------------------------------------------------: | ------------------ |
| 1.左连接                                       |                A  LEFT JOIN  B ON A.KEY=B.KEY                | 左表全保留         |
| 2.右连接(实际多统一用左外连接)                 |               A  RIGHT JOIN  B ON A.KEY=B.KEY                | 右表全保留         |
| 3.内连接                                       |               A INNER JOIN  B  ON  A.KEY=B.KEY               | 既在A表又在B表     |
| 4.左连接加条件筛选                             |     A LEFT JOIN  B  ON  A.KEY=B.KEY  WHERE B.KEY IS NULL     | A表独有信息        |
| 5.右连接在条件筛选                             |    A RIGHT JOIN  B  ON  A.KEY=B.KEY  WHERE A.KEY IS NULL     | B表独有信息        |
| 6.全连接（MySQL 通过左连接和右连接的并集实现） | A  LEFT JOIN  B ON A.KEY=B.KEY  UNION A  RIGHT JOIN  B ON A.KEY=B.KEY | AB表的并集         |
| 7.全连接（4和5并集）                           |                                                              | AB并集去除公共部分 |



## MySQL如何高效插入100w条数据？

## 索引简介

索引定义：Index是帮助MySQL 高效获取数据的数据结构，索引的本质是数据结构。

可以简单理解为排好序的快速查找数据结构

一般来说索引也很大，不可能全部存储在内存中，因此索引往往以索引文件形式存储在磁盘上。

#### 索引的优势：

- 提高数据检索的效率，降低数据库IO成本；

- 通过索引都数据排序，降低数据排序的成本，降低CPU消耗；

#### 索引的劣势：

- 虽然索引提高了查询效率，但同时会降低更新表的速度，如对表进行 insert  update 和delete操作。

因为更新表时MySQL不仅需要保存数据，还需要对索引进行调整，处理因为更新所带来的键值变化后的信息。

- 索引实际上也是一张表，保存了主键和索引字段，索引列也要占用空间。





## Mysql索引结构

### BTree索引 （oracle默认的索引类型）

结点内容：1、关键字  2、指向下一个节点的索引 3、指向记录的指针

结构图

![img](https://img-blog.csdn.net/20170315104705873)

### B+Tree索引

节点内容 1、关键字  2、指向下一个节点的索引 没有指向记录的指针  

B+Tree 更加节约空间，因此一次相同大小的IO，可以加载更多的节点，减少IO消耗，IO也需要时间









### B树和B+树的区别









### 聚簇索引和非聚簇索引

聚簇索引不是单独的索引类型，而是数据的物理存储方法，

主键索引即聚簇索引，非主键索引使用非聚簇索引

![image-20210318112331147](C:\Users\wangdingjian\AppData\Roaming\Typora\typora-user-images\image-20210318112331147.png)

聚簇索引的好处

## MYSQL索引分类

相关语法：

```mysql
-- 创建索引
CREATE [UNIQUE] INDEX [indexname] ON tablename((column));
-- 删除索引
DROP  INDEX  [indexname] ON tablename;
-- 查看索引
SHOW INDEX  FROM tablename；
-- 使用 alter

```



单值索引

一个索引只包含单个列，一个表可以有多个单值索引

```mysql
-- 随表建立索引
CREATE TABLE customer (
    id INT(10) UNSIGNED AUTO_INCREMENT ,
    customer_no VARCHAR(200),
    customer_name VARCHAR(200),
    PRIMARY KEY(id),
    KEY(customer_name),
    UNIQUE(customer_no)
);

```

唯一索引

索引列的值必须唯一，但允许为null

主键索引

设定为主键后数据库回自动建立索引，表的主键一般不会变，防止影响其他表

复合索引

即一个索引包含多个列，设置时必须按预想的顺序来设置 

## 哪些情况需要建立索引

* 主键自动建立唯一索引
* 频繁作为查询条件的字段应该创建索引   where
* 查询中与其他表关联的字段，外键关系建立索引 join
* 单件/组合索引的选择问题，组合索引性价比更高
* 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度order  by
* 查询中统计或者分组字段 group by
* 备注：group by 更需要性能， groupby 隐含了order by ，Mysql8.0之后不在进行排序 

## 哪些情况不需要创建索引

* 表记录数太少
* 经常增删改查的表或者字段
* Where条件里用不到的字段不创建索引
* 过滤性不好的不适合建索引(数据分布比较集中)







# 性能分析

## explain

使用explain关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的，分析你的查询语句或者表结构的性能瓶颈

### 能干什么

* 表的执行顺序

* 哪些表可以使用 

* 那些索引被实际使用

* 表之间的引用

* 每张表有多少行被物理层查询

  

### 怎么使用 Explain+SQL语句

执行计划包含的信息

建表脚本







### 各字段解释

**id**

select 查询的序列表，表名查询中执行select子句或者操作表的顺序

id相同，执行顺序从上到下

id不同 如何是子查询，从大到小，id的序号会递增，id越大优先级越高

id相同不同同时存在

备注：每个单独的id号标识一趟独立的查询，一个SQL的查询次数越少越好



```mysql
explain select * from  t1,t2,t3  where   t1.id=t2.id and  t2.id=t3.id;
```

结果 id均为1 ，按顺序从上到下 t1 t3 t2；

**select_tyep查询类型**

* SIMPLE 简单的select查询 查询中不包含子查询或UNION
* PRIMARY   查询中若包含任何复杂的子部分 ，最外层查询则被标记为Primary
* DERIVED   在FROM表内包含的资产寻被标记为Derived衍生的；
* SUBQUERY  在SELECT或WHERE 列表中包含了子查询

* DEPENDENT SUBQUERY   在SELECT或WHERE 列表中包含了子查询，子查询基于外层（  in   （子查询）  ）;
* UNCACHEABLE SUBQUERY  不可用缓存查询  例如 where  id=@@sort_buffer_size   @@开头是系统变量，随时可能发生变化，一定在缓存内没有。
* UNION  若第二个SELECT 出现在 UNION之后 则被标记为UNION

table  显示这一行数据是关于那张表的

partitions  代表分区表的命中情况，非分区表值为null；

type

* 访问类型排序

  all index range  ref   eq_ref  const，sysytem   null

* 显示了查询使用了何种类型

  从最好到最差依次是

  system>const>eq_ref>ref>range>index>ALL

system  系统内只有一行数据（等于系统表），const类型特例

const 表示通过索引依次就找到了，const用于比较primary key或者unique key  比如where +主键索引

eq_ref  唯一性索引

ref        非唯一性索引

range:只检索给定范围的行 where  id<10

index:出现index是sql使用了索引，但是没用索引进行过滤，一般使用了覆盖索引或者是利用索引来排序分组

all : Full Table Scan  将遍历全表以找到匹配的行 

index_merge  在查询中需要多个索引组合使用       通常出现在有or的关键的sql中

ref_or_null  某个字段筛选需要关联条件又需要null值 

where id=10 or id=null；

index_subquery  子查询索引，利用索引来关联子查询，不在全表扫描

where  id in（select id from t2）;

unique_subquery   唯一子查询索引





possible_keys  可能能用到的索引

key  实际使用的索引 ，查询中若使用了覆盖索引，则该索引和查询的select字段重叠 

key_len 表示索引中使用的字节数，数值越长说明使用到的索引字段越多，可以通过计算查看命中了多少列

![image-20210319105101750](C:\Users\wangdingjian\AppData\Roaming\Typora\typora-user-images\image-20210319105101750.png)



ref  显示索引哪一列被使用了，

rows 整个SQL物理扫描的行数 越少越好，rows是分析出来的，不是真的执行

filtered   表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录的百分比。

:star:Extra 非常重要的额外信息

Using filesort   orderby没使用上索引，手工排序

查询中排序的字段若通过索引去访问将大大提高排序速度，

4.86s→ 0.00s



Using temporary    groupby没用上索引

Using  index

Using  where   

Using  join  buffer  关联字段没用上索引

impossible   where       sql where写错了 

select  tables   optimized away    用上了优化器



# 查询优化

## 如何高效插入100w条数据

JDBC—使用JDBC循环插入

事务 —关闭自动提交，执行完插入语句后一次提交

索引 —删除主键外其余索引

多线程—使用多线程操作 

## 批量数据脚本

往表里插入50w数据

* 建表

* 设置参数

* 创建函数  

* 创建存储过程

* 调用存储过程

  





```mysql
show  variables like 'log_bin_trust_function_creators';
set  global  log_bin_trust_function_creators=1;


```





创建函数— 生成随机员工名称

```mysql

DELIMITER  $$  
CREATE FUNCTION rand_string(n INT) RETURNS  VARCHAR(255)
BEGIN
DECLARE  chars_str  VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
DECLARE  return_str VARCHAR(255) DEFAULT  '';
DECLARE  i  INT   DEFAULT 0;
WHILE   i<n  DO
SET   return_str=CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
SET    i=i+1;

END WHILE;
RETURN  return_str;
END$$
```

创建函数-生成随机员工编号

```mysql
DELIMITER  $$  

CREATE FUNCTION RAND_NUM(from_num INT ,to_num INT)  RETURNS  INT(11)
BEGIN
DECLARE i  INT DEFAULT 0;
SET i=FLOOR(from_num+RAND()*(to_num-from_num+1));

RETURN  i;
END$$

```

创建一个存储过程







执行储存过程

```
 
```









1.查询索引名

show  index from  t_emp;



2.取出索引名

information_schema库中的 statistics表储存了表的储存列  列名 

```mysql
SELECT index_name  FROM  information_schema.`STATISTICS` WHERE table_name='t_emp' AND  index_name<>'PRIMARY' AND  seq_in_index=1;

```

如何循环集合

cursor 游标

FETCH XXX INTO XXX；



3.如何让mysql执行一个字符串

PREPARE  预编译 xxx

EXCUTE 

CALL  proc_drop_index('mydb','t_emp');



```mysql
DELIMITER $$
CREATE  PROCEDURE `proc_drop_index`(dbname VARCHAR(200),tablename VARCHAR(200))
BEGIN
       DECLARE done INT DEFAULT 0;
       DECLARE ct INT DEFAULT 0;
       DECLARE _index VARCHAR(200) DEFAULT '';
       DECLARE _cur CURSOR FOR  SELECT   index_name   FROM information_schema.STATISTICS   WHERE table_schema=dbname AND table_name=tablename AND seq_in_index=1 AND    index_name <>'PRIMARY'  ;
       DECLARE  CONTINUE HANDLER FOR NOT FOUND set done=2 ;      
        OPEN _cur;
        FETCH   _cur INTO _index;
        WHILE  _index<>'' DO 
               SET @str = CONCAT("drop index ",_index," on ",tablename ); 
               PREPARE sql_str FROM @str ;
               EXECUTE  sql_str;
               DEALLOCATE PREPARE sql_str;
               SET _index=''; 
               FETCH   _cur INTO _index; 
        END WHILE;
   CLOSE _cur;
   END$$
```



```
CALL proc_drop_index("dbname","tablename");
```



## 单表索引优化



where 条件2 条件3 条件1   与索引顺序  1 2  3不一致 不影响使用索引，优化器会自动调节顺序；

where 条件1  条件3    只能用上 索引1  

where  条件 2 条件3  用不上索引



### 索引失效

全表扫描——建立索引

全值匹配

:star:最佳左前缀法则

 如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。

不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描



储存引擎不能使用索引范围条件右边的列

——建索引时把常出现范围筛选条件的字段放在最后，比如日期，时间等，where  a=1and b>5 and c=1  只能用上a，b两个索引

mysql 在使用不等于(!= 或者<>)的时候无法使用索引会导致全表扫描

is not null 也无法使用索引,但是is null是可以使用索引的

like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作

字符串不加单引号索引失效

——自动手动类型转换都会导致索引失效

### 单表建立索引的建议

对于单键索引，尽量选择针对当前query过滤性更好的索引

在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。

在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引

在选择组合索引的时候，如果某个字段可能出现范围查询时，尽量把这个字段放在索引次序的最后面

书写sql语句时，尽量避免造成索引失效的情况





