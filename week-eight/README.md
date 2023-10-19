# MySQL锁篇

![一条update语句](.\note\一条update语句.png)

在实际的数据库系统中，每时每刻都在发生锁定，当某个用户在修改一部分数据时，MySQL会通过锁定防止其他用户读取同一数据。

在处理并发读或者写时，通过实现一个由两种类型的锁组成的锁系统来解决问题。两种锁通常被称为**共享锁(shared lock)和排他锁(exclusive lock)，也叫读锁(read lock)和写锁(write lock)**。

读锁是共享的，是互相不阻塞的。多个客户端在同一时刻可以同时读取同一个资源，而不互相干扰。写锁则是排他的，也就是说一个写锁会阻塞其他的写锁和读锁，这是出于安全策略的考虑，只有这样才能确保在给定的时间里，只有一个用户能执行写入，并防止其他用户读取正在写入的同一资源。

## 锁

### 锁分类

按锁粒度分：

全局锁：锁整Database，由MySQL的SQL layer层实现

表级锁：锁某Table，由MySQL的SQL layer层实现

行级锁：锁某Row的索引，也可锁定行索引之间的间隙，由存储引擎实现【InnoDB】

![MySQLServer](.\note\MySQLServer.png)

按锁功能分：

- 共享锁Shared Locks（S锁，也叫读锁）： 为了方便理解，下文我们全部使用读锁来称呼
  - 加了读锁的记录，允许其他事务再加读锁
  - 加锁方式：select…lock in share mode

- 排他锁Exclusive Locks（X锁，也叫写锁）：为了方便理解，下文我们全部使用写锁来称呼
  - 加了写锁的记录，不允许其他事务再加读锁或者写锁
  - 加锁方式：select…for update

### 全局锁

全局锁是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的DML的写语句，DDL语句，已经更新操作的事务提交语句都将被阻塞。其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。

加全局锁的命令为：

```
flush tables with read lock;
```

释放全局锁的命令为：

```
unlock tables;
```

或者断开加锁session的连接，自动释放全局锁

### 表级锁

MySQL的表级锁有四种：

- 表读锁（Table Read Lock）
- 表写锁（Table Write Lock）
- 元数据锁（meta data lock，MDL)
- 自增锁(AUTO-INC Locks)

#### 表读锁、写锁

MySQL 实现的表级锁定的争用状态变量

```sql
# 查看表锁定状态
mysql> show status like 'table%';
```

- table_locks_immediate：产生表级锁定的次数；
- table_locks_waited：出现表级锁定争用而发生等待的次数；

手动增加表锁：

```sql
lock table 表名称 read(write),表名称2 read(write)，其他;
# 举例：
lock table t read; #为表t加读锁
lock table t write; #为表t加写锁
```



# MySQL性能优化篇

