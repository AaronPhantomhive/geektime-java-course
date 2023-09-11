# MySQL架构

## 一条Insert语句

```sql
CREATE TABLE `tab_user` (
	`id` int(11) NOT NULL,
	`name` varchar(100) DEFAULT NULL,
	`age` int(11) NOT NULL,
	`address` varchar(255) DEFAULT NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB;
```

然后向这个表里插入一条数据

```sql
Insert into tab_user(id,name,age,address) values (1,'A',18,'xxx');
```

## Insert语句执行流程

![Insert语句执行流程](.\note\Insert语句执行流程.png)

## 事务

**事务指的是逻辑上的一组操作，组成这组操作的各个单元要么全都成功，要么全都失败。**

**事务作用：保证在一个事务中多次SQL操作要么全都成功，要么全都失败。**

MySQL 是一个服务器／客户端架构的软件，对于同一个服务器来说，可以有若干个客户端与之连接，每个客户端与服务器连接上之后，就可以称之为一个会话（ Session ）。我们可以同时在不同的会话里输入各种语句，这些语句可以作为事务的一部分进行处理。不同的会话可以同时发送请求，也就是说服务器可能同时在处理多个事务，这样子就会导致不同的事务可能同时访问到相同的记录。

事务的隔离性在理论上是指，在某个事务对某个数据进行访问时，其他事务应该进行排队，当该事务提交之后，其他事务才可以继续访问这个数据。但是这样子的话对性能影响太大，所以才会出现各种隔离级别，来最大限度的提升系统并发处理事务的能力，牺牲部分隔离性来提升性能。

### 事务四大特性ACID

数据库事务具有ACID四大特性。ACID是以下4个词的缩写：

- 原子性（Atomicity）： 原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
- 一致性（Consistency）： 事务前后数据的完整性必须保持一致
- 隔离性（Isolation）：多个用户并发访问数据库时，一个用户的事务不能被其它用户的事务所干
  扰，多个并发事务之间数据要相互隔离。隔离性由隔离级别保障！
- 持久性（Durability）： 一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响。

### 事务并发问题

- 脏读：一个事务读到了另一个事务未提交的数据
- 不可重复读：一个事务读到了另一个事务已经提交(update)的数据。引发事务中的多次查询结果不一致
- 虚读 /幻读：一个事务读到了另一个事务已经插入(insert)的数据。导致事务中多次查询的结果不一致

### 隔离级别

- read uncommitted 读未提交【RU】，一个事务读到另一个事务没有提交的数据
  - 存在：3个问题（脏读、不可重复读、幻读）。
- read committed 读已提交【RC】，一个事务读到另一个事务已经提交的数据
  - 存在：2个问题（不可重复读、幻读）。
  - 解决：1个问题（脏读）
- repeatable read:可重复读【RR】，在一个事务中读到的数据始终保持一致，无论另一个事务是否提交
  - 解决：3个问题（脏读、不可重复读、幻读）
- serializable 串行化，同时只能执行一个事务，相当于事务中的单线程
  - 解决：3个问题（脏读、不可重复读、幻读）

#### 安全和性能对比

- 安全性： serializable > repeatable read > read committed > read uncommitted
- 性能 ： serializable < repeatable read < read committed < read uncommitted

#### 常见数据库的默认隔离级别

- MySql： repeatable read
- Oracle： read committed

### 事务底层原理详解

#### 丢失更新问题

两个事务针对同一数据进行修改操作时会丢失更新，这个现象称之为丢失更新问题

#### 解决方案

##### 基于锁并发控制LBCC

使用基于锁的并发控制LBCC（Lock Based Concurrency Control）

##### 基于版本并发控制MVCC

并发控制MVCC（Multi Version Concurrency Control）机制

MVCC使得普通的SELECT请求不加锁，读写不冲突，显著提高了数据库的并发处理能力。MVCC保障了ACID中的隔离性

#### MVCC实现原理【InnoDB】

##### MVCC的定义

> Multiversion concurrency control (MVCC) is a concurrency control method commonly used by database management systems to provide concurrent access to the database and in programming languages to implement transactional memory

MVCC全称叫多版本并发控制，是RDBMS常用的一种并发控制方法，用来对数据库数据进行并发访
问，实现事务。

##### 核心思想

读不加锁，读写不冲突。

##### 实现原理

MVCC 实现原理关键在于数据快照，不同的事务访问不同版本的数据快照，从而实现事务下对数据的隔离级别。

##### 关键要素

MVCC 的实现依赖与Undo日志 与 Read View

**作业题目：什么是 MVCC？**

```
MVCC（Multiversion concurrency control）是多版本并发控制。
是RDBMS常用的一种并发控制方法，用来对数据库数据进行并发访问。
它是一种用于解决数据库读写冲突的技术，通过为每个事务提供一个快照视图（ReadView），并根据一定的规则判断数据的可见性，从而实现不同隔离级别下的一致性读取。
核心思想是读不加锁，读写不冲突。
```

**要点：**

1. Redo 日志

   ```
   Redo日志是一种用于记录数据页修改操作的日志文件，它可以保证在事务提交时，把这些修改行为持久化到磁盘中，以防止系统崩溃后数据丢失。Redo日志记录了事务的行为，可以很好地通过其对页进行回滚操作，也就是undo。
   当我们修改一条数据的时候，会把原来的值写到undo log中，当这条更新语句在事务中执行的时候，事务回滚，就可以通过 undo log将数据恢复成原来的值。
   Undo存放在数据库内部的一个特殊段（segment）中，这个段称为Undo段（undo segment）。
   ```

2. ReadView

   ```
   ReadView是张存储事务id的表，主要包含当前系统中有哪些活跃的读写事务。
   开启事务之后，在第一次查询(select)时，生成ReadView。
   
   其中包含了四个重要的属性：m_ids、min_trx_id、max_trx_id 和 。creator_trx_id。
   m_ids：表示在生成ReadView时，当前系统中活跃的读写事务id列表
   m_low_limit_id：事务id下限，表示当前系统中活跃的读写事务中最小的事务id，m_ids事务列表中的最小事务id
   m_up_limit_id：事务id上限，表示生成ReadView时，系统中应该分配给下一个事务的id值
   m_creator_trx_id：表示生成该ReadView的事务的事务id
   ```

3. 如何判断可见性

   ```
   如果 trx_id < min_trx_id，说明该版本在生成 ReadView 之前已经提交，是可见的；
   如果 trx_id >= max_trx_id，说明该版本在生成 ReadView 之后才出现，是不可见的；
   如果 min_trx_id <= trx_id < max_trx_id，说明该版本在生成 ReadView 时是活跃的，需要进一步判断：
   如果 trx_id 在 m_ids 中，说明该版本还未提交，是不可见的；
   如果 trx_id 不在 m_ids 中，说明该版本已经提交，是可见的。
   ```

